# CORS検証用モックアプリケーション

このプロジェクトは、CORS（Cross-Origin Resource Sharing）とCSP（Content Security Policy）の検証用のモックアプリケーションです。異なるドメイン間での通信テストに使用できます。

## 使用技術

- **Webサーバー**: Apache 2.4
- **コンテナ化**: Docker
- **開発環境**: VS Code + Dev Containers
- **通信**: HTTPS（SSL/TLS）

## 環境構築

### 前提条件

- Docker
- VS Code
- VS Code Remote - Containers 拡張機能

#### VSCode インストール
- link: [Visual Studio Code](https://code.visualstudio.com/)
- 拡張機能 Dev Containers をインストール（[参照](https://blog.kinto-technologies.com/posts/2022-12-10-VSCodeDevContainer/)）
- 拡張機能 WSL をインストール（[参照](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)）

#### WSL インストール, Docker 関連 インストール
- wsl2 の導入、及び docker の wsl2 上へのインストール
    - 参考: [Docker+Windowsの重い問題を解決できそうなWSL2を試してみる](https://tech.griphone.co.jp/2020/08/13/docker-on-wls2-with-intellij/)
    - 「Dockerを入れる」の項は除く。後述の記事を参考のこと。
    - 「docker-compose をセットアップする」の項は除く。後述の記事を参考のこと。
- wsl1 から wsl2 に更新する際にエラーが発生する場合
    - 参考：[WSL2をインストールして使うときの注意点](https://qiita.com/matarillo/items/98d7452967987fe5d633)
- Docker のインストール
    - 参考：[Docker Engin インストール（Ubuntu 向け）](https://matsuand.github.io/docs.docker.jp.onthefly/engine/install/ubuntu/)
    - Docker のインストール後、Docker daemon が起動できない場合のトラブルシューティング
        - Docker のログを確認する  
        ```sudo cat /var/log/docker.log```
        - 以下のようなエラーの場合
            ```
            failed to start daemon: Error initializing network controller: error obtaining controller instance: unable to add return rule in DOCKER-ISOLATION-STAGE-1 chain:  (iptables failed: iptables --wait -A DOCKER-ISOLATION-STAGE-1 -j RETURN: iptables v1.8.7 (nf_tables):  RULE_APPEND failed (No such file or directory): rule in chain DOCKER-ISOLATION-STAGE-1
            (exit status 4))
            ```
        - 以下２つのコマンドを実行し、Docker を再起動する  
        ```sudo update-alternatives --set iptables /usr/sbin/iptables-legacy```  
        ```sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy```

### セットアップ手順

1. リポジトリをクローン
```bash
git clone https://github.com/NorioSaito/CorsTest.git
cd CorsTest
```

2. VS Codeでプロジェクトを開く
```bash
code .
```

3. Dev Containerでコンテナを起動
- VS Codeの左下の「><」アイコンをクリック
- 「Reopen in Container」を選択

4. コンテナの起動を待つ
- 初回起動時は、必要なイメージのダウンロードとビルドが行われます

## 使用方法

1. コンテナが起動したら、ブラウザで以下のURLにアクセス
```
https://localhost
```

2. 自己署名証明書の警告が表示される場合は、開発環境として許可してください

## CORS設定の変更方法

CORSの設定は、Apacheの設定ファイルで管理されています。

### メインアプリケーションのドメインを許可する場合

`apache-config/default-ssl.conf`を編集し、以下のヘッダーを設定します：

```apache
# 特定のドメインを許可する場合
Header set Access-Control-Allow-Origin "https://your-main-app-domain.com"

# または、すべてのドメインを許可する場合（開発環境のみ）
Header set Access-Control-Allow-Origin "*"
```

### 許可するHTTPメソッドの設定

```apache
Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
```

### 許可するヘッダーの設定

```apache
Header set Access-Control-Allow-Headers "Content-Type"
```

### 認証情報の送信を許可する場合

```apache
Header set Access-Control-Allow-Credentials "true"
```

## CSP設定の変更方法

Content Security Policyの設定も、Apacheの設定ファイルで管理されています。

### 基本的なCSP設定

```apache
Header set Content-Security-Policy "default-src 'self' https:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
```

### 特定のドメインからのリソース読み込みを許可する場合

```apache
Header set Content-Security-Policy "default-src 'self' https://trusted-domain.com; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
```

## 設定の反映

設定を変更した場合は、コンテナを再起動する必要があります：

1. VS Codeのコマンドパレット（Ctrl+Shift+P）を開く
2. 「Dev Containers: Rebuild Container」を選択

## 注意事項

- このモックアプリケーションは開発環境でのテスト用です
- 本番環境では、適切なセキュリティ設定を行ってください
- 自己署名証明書は開発環境でのみ使用してください

## 