# はじめてのGitHub Actions CI

CI（継続的インテグレーション）は、変更をこまめに共有し、自動テストなどで問題を早く見つける開発のやり方です。この教材ではPython標準のテスト機能だけを使います。

## 1. プログラムを書く

`calculator.py` の `add(a, b)` は2つの数字を足します。

```python
def add(a, b):
    return a + b
```

## 2. 正しい結果をテストに書く

`test_calculator.py` の `self.assertEqual(add(2, 3), 5)` は「2と3を足した結果が5と等しいか」を検査します。人間が決めた期待値と、実際の結果を比較します。

このフォルダーで実行してください。

```powershell
python -m unittest -v
```

`unittest` が `test_` で始まるファイルを探し、テストを実行します。`OK` なら成功です。

## 3. 自動実行の手順を書く

`.github/workflows/ci.yml` はGitHub Actionsが読む手順書です。リポジトリ直下のこの場所に置く必要があります。

| 設定 | 意味 |
|---|---|
| name | Actions画面に表示する名前 |
| on: push | GitHubへ変更を送ったら起動 |
| on: pull_request | 変更の取り込みを相談するPull Requestの作成・更新などで起動 |
| on: workflow_dispatch | Actions画面から手動でも起動できる |
| permissions: contents: read | コードを読み取る権限を与える |
| jobs: test | testという名前の作業のまとまり |
| runs-on: ubuntu-latest | GitHubが用意するLinux環境で実行 |
| steps | 上から順番に実行する工程 |
| uses: actions/checkout@v7 | リポジトリのコードを実行環境に取得 |
| uses: actions/setup-python@v7 | Pythonを用意。withで3.11を指定 |
| run: python -m unittest -v | 手元と同じテストコマンドを実行 |

`uses` は既製の処理を呼び出し、`run` はコマンドを実行します。`@v7` はActionのバージョンで、Pythonのバージョンではありません。YAMLでは字下げにも意味があるので、スペースを保ってください。

## 4. GitHubに送る

GitHubで空のデモ用リポジトリを作成します。以下は、この教材フォルダーの中で実行する例です。URLは自分のリポジトリに置き換えてください。既存プロジェクトのルートで実行しないでください。

```powershell
git init -b main
git add calculator.py test_calculator.py .github/workflows/ci.yml .gitignore README.md
git commit -m "Add a passing CI demo"
git remote add origin https://github.com/YOUR-NAME/YOUR-DEMO-REPO.git
git push -u origin main
```

- `git add`：記録するファイルを選ぶ。
- `git commit`：変更を手元の履歴に記録する。
- `git remote add`：GitHub上の送信先を登録する。
- `git push`：記録した変更をGitHubへ送る。ここでCIが起動する。

GitとGitHubへの認証が必要です。初回のcommitで名前やメールの設定を求められたら、自分の値をこのリポジトリに設定してください。

## 5. 成功を確認する

リポジトリの **Actions → CI demo → 実行履歴 → test → Run tests** を開きます。テストが通れば緑のチェックと `OK` を確認できます。

変更 → push → GitHubが環境を準備 → コードを取得 → Pythonを準備 → テスト → 結果表示、という流れです。

## 6. わざとバグを入れて失敗させる

`calculator.py` の `return a + b` を `return a - b` に変更します。テストはそのままにします。

```powershell
python -m unittest -v
git add calculator.py
git commit -m "Demo: introduce an addition bug"
git push
```

手元では `AssertionError: -1 != 5` と表示されます。GitHubでも同じテストが実行され、赤い失敗になります。テストコマンドの終了コードが1（失敗）になるため、その工程とジョブが失敗します。

この失敗のpushは、使い捨てのデモ用リポジトリで行う練習です。

## 7. 修正して成功に戻す

`return a + b` に戻します。

```powershell
python -m unittest -v
git add calculator.py
git commit -m "Fix addition"
git push
```

新しい実行が緑になります。前の失敗した履歴は残るので、失敗と修正の対応を確認できます。

## 何がわかったか

CIは、毎回同じ確認を自動で行い、変更による不具合を早く知らせてくれます。ただし、このテストで確認しているのは2 + 3だけです。テストに書いていない動作まで正しいとは保証しません。

この教材は自動テストまでです。本番へ公開する工程はありません。CIが失敗するだけではマージは自動的に禁止されません。チーム開発で成功を必須にするには、別途リポジトリのルール設定が必要です。

## 公式資料

- [GitHub Actionsクイックスタート](https://docs.github.com/en/actions/get-started/quickstart)
- [Python環境を準備する公式Action](https://github.com/actions/setup-python)
