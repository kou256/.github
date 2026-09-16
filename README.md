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
