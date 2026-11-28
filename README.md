# mdx-maas-website

mdx-MaaS ポータルサイトのソースです。GitHub Pages（Jekyll / minima テーマ）で公開します。

公開URL: https://maas.mdx1.jp/ （旧 URL https://toyolabo.github.io/mdx-maas-website/ は 301 で https://maas.mdx1.jp/ に転送されます）

## ページ構成

| ファイル | ページ | 備考 |
| --- | --- | --- |
| `index.md` | mdx-MaaS ユーザーポータル（トップ） | サイトのルート。API の利用案内・申請手順・FAQ |
| `chat.md` | チャットサービス利用案内 | 学認以外のアカウント（mdx Local ID）でチャットを使う方向け。ナビには表示しない（ユーザーポータルの FAQ からのみリンク）。「利用の前提」セクション内に学認ユーザー向けページ（`chat-gakunin.md`）への導線あり |
| `chat-gakunin.md` | チャットサービス利用案内（学認ユーザー向け） | 学認（東京大学・大阪大学など）でチャットを使う方向け。ナビには表示しない（ユーザーポータルの FAQ と `chat.md` からリンク）。学認ログイン URL は `https://sp.maas.mdx1.jp`（学認以外は `https://local.maas.mdx1.jp`） |
| `updates.md` | アップデート履歴 | ナビに表示 |
| `models.md` | 利用可能なモデル一覧 | モデルごとの説明。ユーザーポータル・チャットサービス利用案内・API ドキュメントからリンク |
| `docs/chat-completions.md` | 即時応答 API（Chat Completions API） | API の使い方。ユーザーポータルの「API エンドポイント」からリンク |
| `docs/batch.md` | Batch API（バッチ推論） | 同上 |
| `docs/usage.md` | 使用量確認 API（Usage API） | 同上 |

- `docs/` の API ドキュメントは、以前は GitHub の [mdx-jp/mdx-maas-docs](https://github.com/mdx-jp/mdx-maas-docs) で公開していたものを、このサイトに移したものです。**今後はこのリポジトリの `docs/` を更新してください。** 同じ文章が 2 か所にあると、Google が重複とみなしてどちらか一方しか検索に出さないためです。
- `chat-gakunin.md` は、以前 Google Sites で公開していた「mdx MaaS 学認ユーザー向けポータルサイト」（https://sites.google.com/view/mdx-maas/ ）の内容を 2026-11-27 にこのサイトに移したものです。**今後はこのリポジトリの `chat-gakunin.md` を更新してください。** Google Sites 側は本ページへのリンクだけを残すか、非公開にしてください（同上の重複対策）。

- ナビゲーションに表示するページは `_config.yml` の `header_pages` で指定しています。
- **ページを追加するときは、フロントマターに `description:`（検索結果に出る説明文）を必ず書いてください。** `_config.yml` にはサイト共通の `description` を置いていないため（トップページの `<title>` に連結されてしまうのを避けるため）、書き忘れると検索結果に説明文が出ません。
- FAQ は HTML の `<details>/<summary>` による開閉トグルです（Notion のトグルに相当、JavaScript 不要）。
- フッターは `_includes/footer.html` でテーマ標準のものを置き換え、全ページに運営者（鈴村豊太郎研究室）と問い合わせ先のメールアドレスを表示しています。運営者の表記を変えるときは、こことトップページ（`index.md`）の「mdx-MaaSについて」の両方を直してください。メールアドレスを変えるときは、ここと `index.md` の「問い合わせ先」の両方を直してください。

## 検索エンジン対応

- `/sitemap.xml` は `jekyll-sitemap` プラグイン（`_config.yml` の `plugins`）が自動生成します。対象は HTML ページのみで、画像や CSS は含まれません。
- `robots.txt` はリポジトリ直下のファイルがそのまま公開され、サイトマップの場所を検索エンジンに伝えています。`_config.yml` の `url` を変えたときは `robots.txt` の `Sitemap:` 行も合わせて変更してください。
- 各ページのフロントマターの `last_modified_at`（最終更新日）が `sitemap.xml` の `<lastmod>` になります。**本文を更新したら `last_modified_at` も同じ日付に更新してください。** Google が「内容が変わった」と判断して再クロールする手がかりになります。


## 公開手順（初回のみ）

1. このリポジトリを `main` ブランチに push する
2. GitHub のリポジトリページで **Settings → Pages** を開く
3. **Source** で「**Deploy from a branch**」を選択し、Branch に「**main**」「**/ (root)**」を指定して **Save**
4. 数分待つと https://toyolabo.github.io/mdx-maas-website/ で公開される（ビルド状況はリポジトリの Actions タブで確認可能）
   - 現在はカスタムドメインを設定済み（Settings → Pages → Custom domain = `maas.mdx1.jp`、リポジトリ直下の `CNAME`）のため、実際の公開先は https://maas.mdx1.jp/ です。上記の github.io URL はそこへ 301 で転送されます。

## 公開URLの変更について

`toyolabo.github.io` の部分は GitHub の所有者名、`/mdx-maas-website/` の部分は **リポジトリ名** で決まります。

- パスを変えたい場合: リポジトリ名を変更し、`_config.yml` の `baseurl` を合わせて変更する
- 独自ドメインで公開したい場合: Settings → Pages の Custom domain を設定し、`_config.yml` の `url` を変更・`baseurl` を `""` にする（パスは不要になる）

## 更新のしかた

- **文面の修正**: 各 `.md` ファイルを編集して push するだけで自動で再ビルドされます。
- **アップデート履歴の追加**: `updates.md` の先頭（説明文の下）に日付見出しと箇条書きを追記します。書式はファイル内のコメントを参照してください。
- **利用可能なモデルの追加・変更**: `_data/models.yml` を編集します。モデル一覧ページ（`models.md`）とトップページの表の両方に反映されます。あわせて `models.md` と `index.md` の `last_modified_at` も更新してください。
  - モデル一覧の正式版はこのサイトです。以前使っていた Google スプレッドシートは更新しません。
- **画像の差し替え**: `assets/images/api.png`（ユーザーポータル）と `assets/images/chat.png`（チャット利用案内）を同名で上書きしてください。

## ローカルプレビュー（任意）

GitHub Pages 本番と同じ Ruby 3.3 系＋ github-pages gem で確認できます。
macOS 標準の Ruby (2.6) では動かないため、Homebrew の Ruby 3.3 を使います（初回のみ `brew install ruby@3.3`）。

```sh
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
bundle config set --local path vendor/bundle   # 初回のみ
bundle install                                  # 初回のみ
bundle exec jekyll serve
# → http://127.0.0.1:4000/mdx-maas-website/ をブラウザーで開く
```

ファイルを保存すると自動で再ビルドされます（ブラウザーの再読み込みは手動）。`_config.yml` を変更した場合はサーバーの再起動が必要です。終了は `Ctrl + C`。
