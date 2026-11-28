---
layout: page
title: 埋め込み API（Embeddings API）
description: mdx-MaaS の埋め込み API（OpenAI 互換の Embeddings API）の使い方です。テキストをベクトルに変換して、意味検索・RAG・分類などに使えます。認証、リクエストパラメータ、Python と curl の使用例、エラーへの対応を説明します。
last_modified_at: 2026-11-28
---

mdx-MaaS の **埋め込み API**（OpenAI 互換の Embeddings API）の使い方です。API キーをお持ちでない方は、[ユーザーポータル](../index.md#application-process) から利用を申請してください。

## 概要

テキストを、決まった長さの数値の並び（ベクトル、埋め込み）に変換します。意味の近い文章ほどベクトルも近くなるので、意味検索・RAG（検索拡張生成）・分類・クラスタリング・似た文章の判定などに使えます。

**エンドポイント**: `POST /v1/embeddings`

**ベースURL**: [https://api.maas.mdx1.jp](https://api.maas.mdx1.jp)

**利用できるモデル**:

| モデル名 | ベクトルの次元数 | 1 つの入力の上限 |
|---|---|---|
| `google/embeddinggemma-2` | 768 | 8,192 トークン |

モデルの説明は [利用可能なモデル一覧](../models.md#google-embeddinggemma-2) をご覧ください。

> 埋め込みモデルは、即時応答 API（`/v1/chat/completions`）と Batch API では使えません。反対に、チャット用のモデルを埋め込み API で指定することもできません。どちらも `400 Bad Request` が返ります。

## 認証

すべてのリクエストには、APIキーによる認証が必要です。APIキーはHTTPヘッダーの`Authorization`フィールドにBearer Token形式で指定します。即時応答 API と同じ API キーを使えます。

```
Authorization: Bearer <your-api-key>
```

## 最初のリクエストの待ち時間 {#cold-start}

計算資源を有効に使うため、アクセスがないときはモデルと計算ノードを停止しています。しばらく利用がなかったモデルへの最初のリクエストは起動を待つことになり、応答までに **15〜20 分程度** かかることがあります。起動後は通常の応答速度で利用できます。

**クライアント側のタイムアウトは 30 分程度（1800 秒）に設定することを推奨します。** 既定のタイムアウトが短いクライアントでは、起動を待っている間に接続が切れることがあります。

| クライアント | 既定のタイムアウト | 設定方法 |
|---|---|---|
| OpenAI Python SDK | 10 分 | `OpenAI(timeout=1800)` |
| curl | なし | `--max-time 1800` |

本ドキュメントの使用例には、この設定を入れています。

## リクエストパラメータ

### 必須パラメータ

| パラメータ | 型 | 説明 |
|---|---|---|
| `model` | string | モデル名（例: `google/embeddinggemma-2`） |
| `input` | string または array | ベクトルに変換するテキスト。複数のテキストをまとめて変換するときは、文字列の配列にします。トークン ID の配列（`int[]`、`int[][]`）も指定できます |

### オプションパラメータ {#options}

| パラメータ | 型 | 説明 |
|---|---|---|
| `encoding_format` | string | `"float"`（既定。数値の配列で返す）または `"base64"`（float32 のバイト列を base64 にした文字列で返す） |
| `user` | string | エンドユーザーの識別子（任意） |

> `dimensions`（出力の次元数を減らす指定）には対応していません。指定すると `400 Bad Request` が返ります。次元数を減らしたいときは、受け取ったベクトルの先頭から必要な数（512・256・128 など）を取り出し、長さが 1 になるように正規化し直してください。このモデルは、先頭を切り詰めて使えるように学習されています。

## 入力の先頭に付ける文言 {#prompts}

このモデルは、入力の先頭に用途を表す文言を付けると精度が上がるように学習されています。API はこの文言を自動では付けないので、送る前にテキストに付けてください（出典: [モデルカード](https://huggingface.co/google/embeddinggemma-2)）。`{...}` の部分を実際のテキストに置き換えます。

検索のように、質問（クエリ）と文書で文言が違う用途:

- 検索
  - クエリ: `task: search result | query: {クエリ}`
  - 文書: `title: {タイトル} | text: {本文}`（タイトルが無いときは `title: none | text: {本文}`）
- 質問応答
  - 質問: `task: question answering | query: {質問}`
  - 文書: `title: {タイトル} | text: {本文}`
- 事実確認
  - 主張: `task: fact checking | query: {主張}`
  - 根拠の文書: `title: {タイトル} | text: {本文}`
- コード検索
  - クエリ: `task: code retrieval | query: {クエリ}`
  - コード: `title: {タイトルまたはファイル名} | text: {コード}`

すべての入力に同じ文言を付ける用途:

- 分類: `task: classification | query: {テキスト}`
- クラスタリング: `task: clustering | query: {テキスト}`
- 文の類似度: `task: sentence similarity | query: {テキスト}`

## レスポンス形式

```json
{
  "object": "list",
  "data": [
    {"object": "embedding", "index": 0, "embedding": [0.0681, -0.0292, 0.0319, ...]},
    {"object": "embedding", "index": 1, "embedding": [...]}
  ],
  "model": "google/embeddinggemma-2",
  "usage": {"prompt_tokens": 13, "total_tokens": 13}
}
```

- `embedding` は 768 個の数値の配列です（上の例では途中から省略しています）。
- `data` は、`input` に渡した順に並びます（`index` が入力の位置）。
- 返るベクトルは、長さが 1 になるように正規化されています。そのため、2 つのベクトルの内積がそのままコサイン類似度になります。

## 使用例

### 必要なライブラリのインストール

```bash
pip install openai numpy
```

### 1. テキストをベクトルに変換する（Python）

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.maas.mdx1.jp/v1",
    api_key="your-api-key-here",
    timeout=1800,  # 最初のリクエストは起動を待つことがあるため 30 分
)

response = client.embeddings.create(
    model="google/embeddinggemma-2",
    input=["東京の天気は？", "大阪の名物は？"],
)

print(len(response.data))               # 2（入力の数）
print(len(response.data[0].embedding))  # 768（ベクトルの次元数）
print(response.usage.prompt_tokens)     # 入力のトークン数
```

### 2. テキストをベクトルに変換する（curl）

```bash
curl https://api.maas.mdx1.jp/v1/embeddings \
  --max-time 1800 \
  -H "Authorization: Bearer your-api-key-here" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "google/embeddinggemma-2",
    "input": ["東京の天気は？", "大阪の名物は？"]
  }'
```

### 3. 意味検索（Python）

クエリと文書をそれぞれベクトルに変換し、内積（= コサイン類似度）が大きい順に並べます。先頭に付ける文言は [入力の先頭に付ける文言](#prompts) を参照してください。

```python
import numpy as np
from openai import OpenAI

client = OpenAI(
    base_url="https://api.maas.mdx1.jp/v1",
    api_key="your-api-key-here",
    timeout=1800,
)
MODEL = "google/embeddinggemma-2"

documents = [
    "浅草寺は、東京都台東区にある都内最古の寺院です。",
    "大阪城は、豊臣秀吉が築いた城として知られています。",
    "富士山は、静岡県と山梨県にまたがる日本一高い山です。",
]
query = "東京で古いお寺を見たい"

doc_vectors = np.array([
    d.embedding
    for d in client.embeddings.create(
        model=MODEL,
        input=[f"title: none | text: {t}" for t in documents],
    ).data
])
query_vector = np.array(
    client.embeddings.create(
        model=MODEL,
        input=f"task: search result | query: {query}",
    ).data[0].embedding
)

scores = doc_vectors @ query_vector  # ベクトルは正規化済みなので、内積 = コサイン類似度
for i in np.argsort(-scores):
    print(f"{scores[i]:.3f}  {documents[i]}")
```

大量の文書を変換するときは、1 回のリクエストにまとめる数を数十〜数百件程度にして、何回かに分けて送ってください。

## エラーハンドリング

APIは以下のHTTPステータスコードを返す可能性があります:

### 400 Bad Request

- **原因**: 次のいずれか
    - 指定したモデルが存在しない、または埋め込みに対応していない（チャット用のモデルを指定した）
    - `dimensions` を指定した
    - 入力が長すぎる（1 つの入力が 8,192 トークンを超える）、または `input` が空の配列
- **レスポンス例（チャット用のモデルを指定した場合）**:

    ```json
    {
      "detail": "Model openai/gpt-oss-20b does not support embeddings. Use a model with embedding=true in the model registry."
    }
    ```

### 401 Unauthorized

- **原因**: APIキーが無効、または認証ヘッダーが正しくない
- **対処法**:
    - APIキーが正しく設定されているか確認
    - `Authorization: Bearer <api-key>`形式であることを確認

### 422 Unprocessable Entity

- **原因**: `model` や `input` が無い、または `input` の型が正しくない（`null`、オブジェクト、数値と文字列が混ざった配列など）
- **対処法**: `input` を文字列、文字列の配列、トークン ID の配列のいずれかにする

### 429 Too Many Requests

- **原因**: 1 つの API キーから同時に送っているリクエストの数が上限を超えた
- **レスポンス例**:

    ```json
    {
      "detail": "Too many concurrent requests to google/embeddinggemma-2 from this API key (limit: 20). Please wait for your running requests to finish, then try again."
    }
    ```

- **対処法**: 送っているリクエストが終わるのを待ってから再試行する（レスポンスの `Retry-After` ヘッダーに待つ秒数が入っています）。並列に送る数を減らす

### 500 Internal Server Error

- **原因**: サーバー内部エラー
- **レスポンス例**:

    ```json
    {
      "detail": "Internal server error (request_id: 123e4567-e89b-12d3-a456-426614174000)"
    }
    ```

- **対処法**:
    - しばらく待ってから再試行
    - 問題が続く場合は、`request_id` を添えて管理者に連絡

## 制限事項

- **タイムアウト**: サーバー側のリクエストのタイムアウトは1800秒（30分）
- **入力の長さ**: 1 つの入力は 8,192 トークンまで
- **入力の種類**: テキストだけに対応しています（モデル自体は画像・音声にも対応していますが、このサービスでは未対応です）
- **次元数**: 768 次元で返します（`dimensions` には未対応。減らし方は [オプションパラメータ](#options) を参照）

## 関連ドキュメント

- [利用可能なモデル一覧](../models.md) - 埋め込み API で使えるモデルと、その説明
- [即時応答 API（Chat Completions API）](chat-completions.md) - チャット用のモデルで文章を生成する API
- [Batch API（バッチ推論）](batch.md) - 複数のリクエストをまとめて非同期に処理する API
