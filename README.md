# .github

kou256 アカウント配下の全リポジトリに適用される共通設定。

## 中身

| パス | 役割 |
|---|---|
| `.github/PULL_REQUEST_TEMPLATE.md` | 共通 PR テンプレート（`# 概要` / `# テスト` / `# 補足` / `# 関連 issue`） |
| `.github/ISSUE_TEMPLATE/` | 共通 Issue テンプレート（bug / feature / refactor / chore / docs） |
| `renovate-config.json` | Renovate の共有プリセット |

## 適用のされ方

`.github/` 配下のテンプレートは **default community health files** として、このアカウントが持つ全リポジトリに自動で適用されます。private リポジトリにも適用されます。ただし**適用先リポジトリが自前の同種ファイルを持つ場合は、そちらが優先されます**。特に Issue テンプレートは部分適用がなく、適用先に有効な Issue template か `config.yml` が1つでもあると、ここの `ISSUE_TEMPLATE/` は丸ごと無視されます。

このリポジトリは public である必要があります（private だとこの仕組みが機能しません）。

## ラベル

Issue テンプレートが付けるラベルは、**適用先の各リポジトリに実体として存在する必要があります**。テンプレートだけでは自動作成されません。

```bash
gh label clone kou256/template --repo kou256/<name> --force
```

使うラベルは `bug` / `feature` / `refactor` / `chore` / `documentation` の5種と、運用用の `wontfix` / `duplicate` です。

## Renovate

`renovate-config.json` は共有プリセットの実体です。各リポジトリの `renovate.json` から次のように参照します。

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>kou256/.github:renovate-config"]
}
```

Renovate はオンボーディング時にこのファイルを自動検出します。Renovate App をこのリポジトリにもインストールしておく必要があります。

### 自動マージ

| 更新 | PR を出すタイミング | マージ |
|---|---|---|
| patch / minor（脆弱性以外） | 毎週月曜 9:00 前（UTC。`timezone` は未設定） | CI のステータスチェックがすべて成功したあと、Renovate が PR をマージする |
| major（脆弱性以外） | 同上。ただし Dependency Dashboard で承認するまで PR は作られない | 自動マージしない |
| 脆弱性の patch / minor | schedule を待たず、検知後すぐ。グループ「non-major dependencies」にも入れない | 上と同じく、チェック成功後に自動マージ |
| 脆弱性の major | 上と同じくすぐ PR を出す（Dashboard 承認は不要） | 自動マージしない |

自動マージは GitHub の Auto-merge（`platformAutomerge`）ではなく、Renovate が次回実行時に PR をマージする（`automergeType: "pr"`、`platformAutomerge: false`）。即時ではない。チェックが付いていない、失敗している、まだ走っている、のいずれかならマージしない。

各リポジトリ側で必要な設定:

- Renovate の PR に対して成功するステータスチェック（CI）があること。チェックが無いリポジトリはマージされない。
- Renovate App にその PR をマージできる権限があること。ブランチ保護でレビュー必須にしている場合は、Renovate App を必須レビューのバイパス対象に入れないと自動マージは止まる。
- 脆弱性 PR を出すには、Dependency graph と Dependabot alerts を有効にし、Renovate App に Dependabot alerts の読み取り権限が必要。

`platformAutomerge` を `true` に変える場合は、各リポジトリで GitHub の Allow auto-merge を有効にし、ブランチ保護の「Require status checks to pass before merging」で必須チェックを1つ以上選ぶこと。必須チェックが無いと、CI が走る前や失敗した状態でもマージされ得る。

### コミットメッセージ

コミットメッセージと PR タイトルは、種類 `deps` と更新の絵文字 `⬆️` を組み合わせる。説明は日本語の体言止めにする。パッケージ名の大文字小文字は維持する（`commitMessageLowerCase: "never"`）。

`config:recommended` は本番依存を `fix(deps):`、それ以外を `chore(deps):` にする。このプリセットはそれを上書きし、依存関係の変更はすべて `deps:` にする。

| 更新 | タイトル |
|---|---|
| 個別の minor / patch | `deps: ⬆️ eslint を v9.14.0 に更新` |
| 個別の major | `deps: ⬆️ webpack を v5 に更新`（メジャー番号のみ。同一メジャー内の追従でタイトルを変えない） |
| patch / minor を複数まとめたとき | `deps: ⬆️ 非メジャー依存関係の更新` |
| それ以外のグループ（複数件） | `deps: ⬆️ <groupName> の更新` |
| ダイジェスト更新 | `deps: ⬆️ node のダイジェストを 1a2b3c4 に更新` |
| ピン（1件） | `deps: ⬆️ lodash を v4.17.21 に固定` |
| ピン（複数） | `deps: ⬆️ 依存関係のバージョン固定` |
| ロックファイルメンテナンス | `deps: ⬆️ ロックファイルのメンテナンス` |
| ロールバック | `deps: ⬆️ eslint を v8.57.0 へロールバック` |
| パッケージ置換 | `deps: ⬆️ left-pad を string-width v5.1.2 に置換` |
| 脆弱性 | 上記の末尾に `（脆弱性）`。例: `deps: ⬆️ minimist を v1.2.8 に更新（脆弱性）` |

グループに属していても、そのブランチの依存が1件だけのときは個別のタイトルになる。

依存関係の追加（`deps: ➕`）と削除（`deps: ➖`）に当たる更新種別は Renovate にない。置換も依存関係の更新として `deps: ⬆️` を使う。
