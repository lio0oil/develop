# django

windows環境のVSCodeでdjango環境を構築する。

## quick install guide

```PowerShell
#ctrl+shift+@で新規ターミナル
python -m venv .env
.\.env\Scripts\activate

#ctrl+shift+pでコマンドパレットを開いて>python: select interpreter
#.\.env\Scripts\python.exeを選択

pip install django

#フォーマッター
pip install black
#コードチェッカー
pip install flake8

#.vscode以下のsettings.jsonに以下をコピペ
```

```json
{
    // リンタの設定
    "python.linting.pylintEnabled": false,
    "python.linting.flake8Enabled": true,
    "python.linting.lintOnSave": true,
    "python.linting.flake8Args": [
        "--ignore=E203,E501,W503,W504"
    ],
    // フォーマッタの設定
    "python.formatting.provider": "black",
    "editor.formatOnSave": true,
    "editor.formatOnPaste": false,
    // 改行コード
    "files.eol": "\n"
}
```

```PowerShell

#djangoの開発スタート
django-admin startproject config .

#ctrl+shift+dでデバッグに移動し、実行とデバッグボタンを押しpython->djangoを選択
#http://127.0.0.1:8000/ にアクセスでロケットの画面

```

以降、pollsアプリケーションを作成するの手順
<https://docs.djangoproject.com/ja/3.2/intro/install/>