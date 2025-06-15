FROM node:22-alpine
LABEL name="project my-microservice builder"

WORKDIR /opt/build

RUN apk add --update \
&& apk add --no-cache ca-certificates \
&& apk add --no-cache --virtual .build-deps curl git python3 alpine-sdk

# 全プロジェクトファイルをコピー
COPY . .

# 依存関係をインストール
RUN yarn install

# プロジェクトをビルド
RUN yarn workspaces foreach --all run build

# シンプルにNode.jsのモジュール解決を使用するために、package.jsonのパスベースのimportを準備
WORKDIR /opt/app
RUN mkdir -p packages/example
RUN mkdir -p services/my-microservice

# バンドルしたpackage.jsonを作成
RUN echo '{"name":"project-bundle","type":"module","dependencies":{"@project/example":"file:packages/example"}}' > package.json

# exampleパッケージをコピー
RUN cp -r /opt/build/packages/example/dist ./packages/example/dist
RUN cp /opt/build/packages/example/package.json ./packages/example/

# my-microserviceサービス設定とコード
RUN cp -r /opt/build/services/my-microservice/dist ./services/my-microservice/dist
RUN cp /opt/build/services/my-microservice/package.json ./services/my-microservice/

# 依存関係をインストール（file:パスの参照を解決）
RUN npm install

FROM node:22-alpine
LABEL name="project my-microservice"
LABEL version="0.1.0"

WORKDIR /usr/app

# アプリケーションをコピー
COPY --from=0 /opt/app ./

# アプリケーションを実行
CMD ["node", "--enable-source-maps", "services/my-microservice/dist/index.js"]
