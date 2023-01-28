# WSL2 Docker

## 手順

### Rancher Desktopのインストール

<https://rancherdesktop.io/>

Setting

- WSL : Ubuntu
- Container Engine : dockered
- Kubernetes : Disabled


### WSL2のUbuntuをインストール

Windows Terminalを管理者で実行

```PowerShell
wsl --install -d Ubuntu
```

### WSL2でDockerを起動する

```PowerShell
#Dockerレジストリにログイン
docker login

#Docker Desktopを事前にインストールしていて以下エラーが発生した場合コンフィグ削除
#error getting credentials - err: docker-credential-desktop.exe resolves to executable in current directory (./docker-credential-desktop.exe), out: ``
cd .docker/
rm config.json 

#docker起動
docker-compose up -d

#docker-composeの名称を変更している場合
docker-compose -f docker-compose_mysql80.yaml up -d
```

windowsのホストからはlocalhostでアクセスできる

## メモ

### WSL --shutdown した時

```PowerShell
#Rancher Desktopの再起動
rdctl shutdown
rdctl start --path "C:\Program Files\Rancher Desktop\Rancher Desktop.exe"
```

### VPN?

<https://github.com/rancher-sandbox/rancher-desktop/issues/1899#issuecomment-1109128277>
