FROM kong/kong-gateway:3.10

# 必要なプラグインを事前インストール（カスタム/公式含む）
# RUN luarocks install kong-plugin-xxx

# 環境変数や構成ファイル（kong.conf）をCOPYしておく
# COPY kong.conf /etc/kong/kong.conf
ENV KONG_LOG_LEVEL=info

# Kongユーザのセキュリティ強化
USER kong
