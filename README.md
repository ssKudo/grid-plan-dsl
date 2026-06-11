# テキストで育てる建築図面の共有記憶

**Agent AI と対話しながら、住宅図面をテキストで記述し、ASCII 図と法規チェック用データへ変換するための軽量 DSL。**

本プロジェクトは、図面を直接 CAD のように描くのではなく、実務者が読み取った建物情報を `通り芯 / 部屋 / 開口 / 設備` としてテキストで宣言し、Agent AI がそれを構造化する「対話型テキスト製図」の研究メモです。

最終的には、建築法規チェックエンジンが扱える `building-card.yaml` を生成し、PDF OCR は正解の自動抽出ではなく、入力済みモデルの根拠照合に使うことを目指します。

## 核心アイデア

従来の建築法規自動判定では、PDF 図面や BIM/IFC から情報を自動抽出するアプローチが多く取られます。しかし、PDF OCR の誤認識や図面表記のばらつきは、寸法・面積・開口条件のように小さな差が結果を左右する領域では大きなリスクになります。

この DSL は、その順序を反転します。

**人間が図面を読み、Agent に説明する。Agent が建物モデルへ構造化する。PDF OCR は、そのモデルが図面上の記載と整合しているかを検証する。**

つまり、問題を「PDF からの完全自動抽出」ではなく、「人間が入力した構造化モデルの検証」に変換します。

## このリポジトリの位置づけ

このリポジトリは、現時点では実装済みアプリケーションではなく、以下を固定するための研究ノートです。

- Agent AI と人間が共有できる、住宅平面図の最小語彙
- 通り芯グリッドに基づく `node / edge / cell` モデル
- 自然文指示から YAML と ASCII 図へ変換するワークフロー
- 法規チェック前処理としての `building-card.yaml` の形
- PDF OCR を「抽出」ではなく「根拠照合」に使う設計方針
- Hermes Agent のような自己改善型エージェントへ注入する建築ドメイン記憶の形
- GitHubリポジトリを、設計・確認申請・積算・施工で共有するプロジェクト記憶として扱う考え方

## Hermes との接続方針

このプロジェクトは、Hermes Agent のような自己改善型エージェントに対して、建築ドメインの記憶・スキル・ワークフローを注入するためのレイヤーとして使うことを想定しています。

Hermes が提供するのは、記憶、スキル、ツール、セッション、定期実行、過去会話の参照といった汎用エージェントのループです。Grid Plan DSL はその上に、建物記憶、図面読解スキル、根拠照合、未解決事項管理、建築確認準備のワークフローを載せるための建築レイヤーです。

最初の目標は Hermes 本体のフォークではなく、Hermes skill bundle として建築確認向けの振る舞いを追加することです。詳細は [agent-architecture.md](./agent-architecture.md)、[hermes-skills.md](./hermes-skills.md)、[hermes-skills/grid-plan-dsl/SKILL.md](./hermes-skills/grid-plan-dsl/SKILL.md) に整理しています。

将来的には、設計者、確認申請担当、積算担当、現場監督が同じGitHubリポジトリをクローンし、それぞれの役割に応じたエージェントスキルで同じ建物記憶を引き継ぐことを想定しています。後工程ほど、ゼロから細かく入力するのではなく、既に育った `building-card.yaml` を確認・補足・修正する使い方になります。

## 用語

- **Grid**: 通り芯の集合。X 方向を数字、Y 方向をかなで表す。
- **Node**: 通り芯の交点。
- **Edge**: 隣接 Node を結ぶ線分。壁、開口、建具、境界を載せる。
- **Cell**: 4 つの Edge に囲まれた最小区画。部屋、設備、用途を載せる。
- **Building Card**: 法規チェックに渡すための決定的な建物データ。
- **Evidence Check**: Building Card の値が PDF 図面上の記載と一致するかを照合する工程。

## グリッドの意味づけ

日本の木造住宅で一般的な通り芯を共通言語として採用します。

```text
Y軸: い, ろ, は, に, ...
X軸: 1, 2, 3, 4, ...
Cell: い-1, い-2, ろ-1, ...
Edge: "い通り 1-2間", "2通り い-ろ間", ...
```

このリポジトリでは、`い-1` のような Cell ID は「い通りからろ通り、1通りから2通りに囲まれる区画」を指すものとして扱います。つまり、Cell は左下側の Y/X ラベルで代表します。

## データモデル例

### 外形

```yaml
outline:
  unit: mm
  origin: southwest
  path:
    - E 11890
    - N 3000
    - W 5000
    - N 4600
    - W 6890
    - S 7600
    - close
```

### グリッド

```yaml
grid:
  module: 910
  x: [1, 2, 3, 4, 5, 6, 7]
  y: [い, ろ, は, に, ほ, ま]
```

### 部屋

```yaml
rooms:
  - id: ldk
    name: LDK
    type: habitable_room
    cells: [に-4, に-5, に-6, ほ-4, ほ-5, ほ-6]
```

### 開口・建具

```yaml
openings:
  - id: w1
    type: window
    template: 16520
    edge: "い通り 1-2間"
    room: washitsu
```

## 最小ケーススタディ

### 1. 自然文入力

```text
910 モジュール。X は 1 から 7、Y は い、ろ、は、に、ほ、ま。
和室は い-1 から は-3 まで。
LDK は に-4 から ほ-6 まで。
洗面は ほ-1 と ほ-2。
い通り 1-2 間に和室の窓 16520。
```

### 2. 構造化された Building Card

```yaml
project:
  name: sample-house
  use: detached_house
  structure: wood

grid:
  unit: mm
  module: 910
  x: [1, 2, 3, 4, 5, 6, 7]
  y: [い, ろ, は, に, ほ, ま]

rooms:
  - id: washitsu
    name: 和室
    type: habitable_room
    cells: [い-1, い-2, い-3, ろ-1, ろ-2, ろ-3, は-1, は-2, は-3]
  - id: ldk
    name: LDK
    type: habitable_room
    cells: [に-4, に-5, に-6, ほ-4, ほ-5, ほ-6]

openings:
  - id: w1
    type: window
    template: 16520
    edge: "い通り 1-2間"
    room: washitsu
```

### 3. ASCII フィードバック

```text
Legend: WA = 和室, LD = LDK, SE = 洗面, w = 窓

     1    2    3    4    5    6    7
ま   +----+----+----+----+----+----+
     |SE  |SE  |    |LD  |LD  |LD  |
ほ   +----+----+----+----+----+----+
     |    |    |    |LD  |LD  |LD  |
に   +----+----+----+----+----+----+
     |WA  |WA  |WA  |    |    |    |
は   +----+----+----+----+----+----+
     |WA  |WA  |WA  |    |    |    |
ろ   +----+----+----+----+----+----+
     |WA  |WA  |WA  |    |    |    |
い   +--w-+----+----+----+----+----+
```

ユーザーはこの ASCII 図を見て、「LDK は に-4 から ま-6 まで」「窓は 2-3 間だった」のように修正できます。Agent は修正内容を YAML に反映し、再度 ASCII 図を出力します。

## ワークフロー

1. **対話構築**: `外形` -> `グリッド` -> `部屋・設備` -> `開口・建具` の順に Agent へ指示する。
2. **構造化**: Agent が `building-card.yaml` を生成する。
3. **視覚確認**: ASCII 平面図で認識状態を確認する。
4. **修正ループ**: 自然文の修正指示を YAML と ASCII 図へ反映する。
5. **根拠照合**: PDF OCR を使い、面積・室名・建具記号などの根拠を確認する。
6. **法規判定**: 決定的な入力データに基づき、OK/NG/不足情報を出力する。

## リポジトリ構成

- [theory.md](./theory.md): コンセプト、目的、データモデルの研究ノート。
- [workflow.md](./workflow.md): 実務的な運用フローとデータ構造の定義。
- [examples/building-card.yaml](./examples/building-card.yaml): 最小 Building Card 例。
- [agent-architecture.md](./agent-architecture.md): Hermes型エージェントに接続する全体構想。
- [hermes-skills.md](./hermes-skills.md): 建築確認向けHermesスキルバンドル案。
- [hermes-skills/grid-plan-dsl/SKILL.md](./hermes-skills/grid-plan-dsl/SKILL.md): Hermes skill雛形。
- [github-workflow.md](./github-workflow.md): GitHubを建築プロジェクト記憶として使う構想。

## 今後の展望

- YAML スキーマとバリデータの定義
- ASCII 平面図レンダラの実装
- Hermes skill bundle としてのインストール手順
- ロール別スキル（設計者、確認申請、積算、現場監督）の整理
- GitHubブランチ/PR/Issueを使った引き継ぎワークフロー
- 採光・換気・排煙などの法規チェックルールの YAML 化
- PDF 図面上の根拠位置と Building Card のリンク
- 直交グリッドでは表しにくい斜め壁・不整形平面への拡張
- Z 軸方向の断面グリッドによる高さ・屋根・斜線制限表現
- 将来的なターミナルメニュー/TUI
