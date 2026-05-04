# yamamoto-rag-x

山本さんの RAG（`insights/`）と X（Twitter）リサーチを組み合わせ、**長文 X 記事** または **YouTube 台本（30分＋Shorts3本）** を生成する [Claude Code](https://claude.ai/claude-code) 用プロジェクトです。

| 用途 | スキル定義（正本） |
|------|---------------------|
| X「記事」向け長文 | `.claude/skills/yamamoto-x-post/SKILL.md` |
| YouTube 台本 | `.claude/skills/yamamoto-yt-script/SKILL.md` |

手順・プロンプト・パスは各 `SKILL.md` を優先してください。こちらの README は概要と X API セットアップです。

---

## 機能（ワークフロー概要）

### X 記事（`yamamoto-x-post`）

1. テーマ受け取り → X リサーチ ON/OFF、X API の有無を確認  
2. X でリサーチ（任意）→ WebSearch → Nitter → X API のフォールバック  
3. 記事構成の整理（リサーチ OFF ならスキップ可）  
4. **RAG** … **`insights/_routing.md` を先に読み**、主カテゴリ1・補助0〜1で仕訳し、優先ファイル 5〜8 件を読む。足りなければ `insights/*.txt` を Grep  
5. 長文 **1 本**を生成 → 確定後、`articles/` に保存  

### YouTube 台本（`yamamoto-yt-script`）

1〜3 は記事スキルと同様の流れ（台本構成ステップあり）。  
4. RAG は同じく **`_routing.md` 優先** 。本編に反映する具体事例は **2〜3 件**、Shorts はその事例の別角度での使い回し。  
5. `insights/video_prompt/Louis Gleeson氏プロンプト.md` を読み込み、台本を生成 → `scripts/` に保存。  

X 記事の途中から **「YouTube台本も作る」** で続けると、リサーチ・RAG を引き継いで台本ステップに進めます（各 SKILL の説明どおり）。

各ステップではスキル内の承認 UI（選択肢）を想定しています。

---

## 生成物の仕様（概要）

| 項目 | X 記事 | YouTube 台本 |
|------|--------|----------------|
| 形式 | X「記事」用 Markdown | Markdown（本編 9,000〜10,000 字目安＋Shorts3本） |
| 分量・構成 | 約 2,000〜3,000 文字、見出し 7〜12 | 12 セクション構成（詳細はプロンプト） |
| 文体 | タメ口・独り言調。ハッシュタグ・絵文字なし | 話し言葉（プロンプトの BRAND DNA に従う） |
| 保存先 | `articles/` | `scripts/`（ファイル名例: `*_yt.md`） |

---

## プロジェクト構成（現状）

```
yamamoto-rag-x/
├── README.md
├── articles/                    ← 生成した X 記事
├── scripts/                     ← 生成した YouTube 台本
├── insights/
│   ├── _routing.md              ← RAG 仕訳（grep より先に必ず読む）
│   ├── video_prompt/            ← 台本用プロンプト（Louis Gleeson）
│   └── *_analysis.txt           ← RAG 用分析テキスト（約 50 本）
└── .claude/skills/
    ├── yamamoto-x-post/SKILL.md
    └── yamamoto-yt-script/SKILL.md
```

**運用上の整理:** 以前の `insights/_used/`（採用履歴）は廃止済みです。リポジトリ側で「使用済み」管理はしません。

---

## データ運用の原則

| 観点 | ルール |
|------|--------|
| 仕訳 | **RAG 検索より先に `insights/_routing.md`** を読み、主カテゴリ1・補助0〜1で絞る |
| 取得 | 1 テーマあたり **広めに 5〜10 ファイル** 読んでもよい |
| 反映 | X 記事: **1〜2 件** / 台本本編: **2〜3 件**（Shorts は本編事例の使い回し） |
| 元データ | `insights/` のファイルは移動・削除しない |
| 再利用 | 同一 `*_analysis.txt` の頻度・切り口はユーザーの指示に従う（リポ側にクールダウンログは持たない） |

新規の `*_analysis.txt` を増やしたら、**`_routing.md` にカテゴリ・キーワード・優先ファイルを追記**すると選定が安定します。

---

## ローカルパスについて

`SKILL.md` 内の **`insights` 等のパスは絶対パス**で書かれています。**クローン先やマシンが違う場合は、自分の環境のパスに合わせて編集**してください。

---

## 必要な環境

- [Claude Code](https://claude.ai/claude-code) がインストール済みであること  
- インターネット接続（X リサーチ用）  

---

## セットアップ

### 1. リポジトリをクローン

```bash
git clone https://github.com/ai776/yamamoto-rag-x2.git
cd yamamoto-rag-x2
```

（フォルダ名は任意です。合わせて `.claude/skills/*/SKILL.md` のパスも更新してください。）

### 2. X API の設定（任意）

X API がなくてもスキルは動作します（WebSearch + Nitter 等）。  
設定すると検索精度が上がります。

---

## X API セットアップ手順

### 料金

| プラン | 料金 | 備考 |
|--------|------|------|
| Pay-Per-Use（従量課金） | 最低 **$5（約750円）** から | 2026年2月〜 |
| ~~Free（無料）~~ | ~~$0~~ | 2026年2月に廃止。503 等の原因になりやすい |

### 前提条件

- 有効な X(Twitter) アカウント  
- クレジットカード  

### STEP 1: Developer Portal でアカウント作成 & Bearer Token 取得

1. **https://developer.x.com** にサインイン  
2. アカウント名・利用目的（**250文字以上**）を入力して送信  
3. **「Projects & Apps」** → 自分の App → **「Keys and tokens」** → **Bearer Token** を Generate してコピー  

> Bearer Token は再表示されないことが多いです。失ったら Regenerate してください。

### STEP 2: Developer Console で Pay-Per-Use に切り替え

1. **https://console.x.com** を開く（STEP 1 と別ページ）  
2. **「アプリ」** で App が **Free** の場合は **Pay-Per-Use** に変更  

> Free のままだと検索 API で `403` などになりやすいです。

### STEP 3: クレジット購入 & 支出上限

1. **「請求書作成」→「クレジット」** で $5 などをチャージ  
2. **支出上限**を設定（無制限のままにしない）  

### STEP 4: 環境変数

```bash
echo 'export X_API_BEARER_TOKEN="ここにBearer Tokenを貼り付け"' >> ~/.bashrc
source ~/.bashrc
```

macOS（zsh）の場合は `~/.zshrc` が一般的です。

### STEP 5: 動作確認

```bash
echo $X_API_BEARER_TOKEN

curl -s -H "Authorization: Bearer ${X_API_BEARER_TOKEN}" \
  "https://api.x.com/2/tweets/search/recent?query=test&max_results=10" \
  | python3 -m json.tool | head -20
```

`"data"` が返れば成功のことが多いです。

---

## トラブルシューティング

| エラー | 原因の例 | 対応 |
|--------|-----------|------|
| `403 Client Forbidden` | App が Free のまま | Pay-Per-Use に切り替え |
| `503 Service Unavailable` | 旧 Free キー等 | Pay-Per-Use へ移行・キー再確認 |
| `401 Unauthorized` | Token 不正・期限 | Regenerate して環境変数を更新 |

---

## 使い方（例）

Claude Code で、テーマと「X 記事を書いて」「YouTube 台本を」などと依頼するとスキルが起動します。

```
外注の記事でXの長文を書いて
```

---

## 参考リンク

- [X Developer Portal](https://developer.x.com)  
- [X Developer Console](https://console.x.com)  
- [X API セットアップ詳細ガイド](https://www.and-and.co.jp/gas-lab/x-twitter-api-start/)  
- [503 解決（Pay-Per-Use）](https://www.and-and.co.jp/gas-lab/x-twitter-api-503-error/)  
- [X API 料金](https://docs.x.com/x-api/getting-started/pricing)  

---

## ライセンス

Private - All Rights Reserved
