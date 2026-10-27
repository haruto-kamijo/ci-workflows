# ci-workflows

自分のリポジトリで共通して使う CI（reusable workflow）。

## 使い方

新しいリポジトリは [repo-template](https://github.com/haruto-kamijo/repo-template) から作れば、最初から入っている。

```sh
gh repo create <name> --private --template haruto-kamijo/repo-template --clone
```

既存のリポジトリに入れる場合は、`repo-template` の `.github/workflows/ci.yml` をコピーする。

## 判定のしかた

`detect` ジョブが次の 2 つを判定し、必要なジョブだけを実行する。

1. **構成**：深さ 3 までのフォルダにある `package.json`（Web）と `pyproject.toml` / `requirements.txt` / `setup.py`（Python）を探す。見つかったフォルダごとにジョブを実行する
2. **変更ファイル**：PR や push で変わったファイルの種類

| ジョブ | 実行される条件 | 内容 |
|---|---|---|
| web | `package.json` があり、JS/TS/CSS や依存関係が変わった | `lint` → `tsc --noEmit`（`tsconfig.json` がある場合）→ `test` → `build`。スクリプトが無いものは飛ばす。npm / pnpm / yarn はロックファイルで判定 |
| python | Python のプロジェクトがあり、`.py` や依存関係が変わった | `ruff check` → `ruff format --check` → `pytest`（テストがある場合） |
| docs | `.md` / `.mdx` が変わった | markdownlint。設定ファイルが無ければ、日本語で誤検出しやすい行の長さなどのルールを外した既定設定を使う |
| workflows | `.github/workflows/` が変わった | actionlint |

手動実行（`workflow_dispatch`）では `run-all: true` になり、変更に関係なくすべて実行する。

## 入力

| 名前 | 既定値 | 説明 |
|---|---|---|
| `node-version` | `20` | `.nvmrc` / `.node-version` が無い場合の Node.js |
| `python-version` | `3.12` | `.python-version` が無い場合の Python |
| `run-build` | `true` | ビルドに秘密の環境変数が要るプロジェクトでは `false` |
| `run-all` | `false` | 変更ファイルに関係なくすべて実行する |

## 注意

- このリポジトリは公開している。非公開のままだと、公開リポジトリ（PortfolioSite など）から呼び出せないため。秘密情報は置かないこと
- 呼び出し側のジョブには `contents: read` と `pull-requests: read` の権限が必要
- 中身を変えると、`@main` で呼び出している全リポジトリに即座に反映される
