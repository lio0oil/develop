# CDKの手順

```PowerShell

cd wordpress_cdk

# CDKプロジェクトを作成
cdk init --language python

```

```PowerShell

.venv\Scripts\activate.bat

pip install -r requirements.txt

```

```PowerShell
# CloudFormation形式のテンプレートファイルに変換
cdk synth

# AWSへデプロイ
cdk deploy

# スタックを削除
cdk destroy

```
