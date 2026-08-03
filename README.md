# mdx-maas-website

mdx MaaS ポータルサイトのソースです。GitHub Pages（Jekyll / minima テーマ）で公開します。

公開URL: https://toyolabo.github.io/mdx-maas-website/

## ページ構成

| ファイル | ページ | 備考 |
| --- | --- | --- |
| `index.md` | mdx MaaS ポータル（トップ） | 学認ユーザー / 個別アカウントユーザーの振り分けページ |
| `portal.md` | mdx MaaS ユーザーポータル | ナビには表示しない（トップページからリンク） |
| `chat.md` | チャットサービス利用案内 | ナビには表示しない（ユーザーポータルの FAQ からのみリンク） |
| `updates.md` | アップデート履歴 | ナビに表示 |

- ナビゲーションに表示するページは `_config.yml` の `header_pages` で指定しています。
- FAQ は HTML の `<details>/<summary>` による開閉トグルです（Notion のトグルに相当、JavaScript 不要）。
- テーマ標準のフッターは `_includes/footer.html` の空ファイルで非表示にしています。

## 公開手順（初回のみ）

1. このリポジトリを `main` ブランチに push する
2. GitHub のリポジトリページで **Settings → Pages** を開く
3. **Source** で「**Deploy from a branch**」を選択し、Branch に「**main**」「**/ (root)**」を指定して **Save**
4. 数分待つと https://toyolabo.github.io/mdx-maas-website/ で公開される（ビルド状況はリポジトリの Actions タブで確認可能）

## 公開URLの変更について

`toyolabo.github.io` の部分は GitHub の所有者名、`/mdx-maas-website/` の部分は **リポジトリ名** で決まります。

- パスを変えたい場合: リポジトリ名を変更し、`_config.yml` の `baseurl` を合わせて変更する
- 独自ドメインで公開したい場合: Settings → Pages の Custom domain を設定し、`_config.yml` の `url` を変更・`baseurl` を `""` にする（パスは不要になる）

## 更新のしかた

- **文面の修正**: 各 `.md` ファイルを編集して push するだけで自動で再ビルドされます。
- **アップデート履歴の追加**: `updates.md` の先頭（説明文の下）に日付見出しと箇条書きを追記します。書式はファイル内のコメントを参照してください。
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
