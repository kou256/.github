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
