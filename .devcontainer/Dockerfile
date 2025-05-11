FROM debian:bookworm

# 必要なパッケージのインストール
RUN apt-get update && \
    apt-get install -y \
    apache2 \
    openssl \
    && rm -rf /var/lib/apt/lists/*

# SSL証明書の生成
RUN mkdir -p /etc/ssl/private && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/apache-selfsigned.key \
    -out /etc/ssl/certs/apache-selfsigned.crt \
    -subj "/C=JP/ST=Tokyo/L=Tokyo/O=Development/CN=localhost"

# Apache設定ファイルのコピー
COPY apache-config/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf

# テスト用HTMLファイルのコピー
COPY html/index.html /var/www/html/

# SSLモジュールとヘッダーモジュールを有効化
RUN a2enmod ssl && \
    a2enmod headers && \
    a2ensite default-ssl

EXPOSE 443

# Apacheをフォアグラウンドで実行
CMD ["apache2ctl", "-D", "FOREGROUND"] 