# building container
FROM node:22-alpine

ENV PACKAGE_NAME=example
ENV SERVICE_NAME=my-microservice

WORKDIR /opt/build

RUN apk add --update \
&& apk add --no-cache ca-certificates \
&& apk add --no-cache --virtual .build-deps curl git python3 alpine-sdk

COPY . .

RUN yarn install

RUN yarn workspaces foreach --all run build

WORKDIR /opt/app
RUN mkdir -p packages/${PACKAGE_NAME}
RUN mkdir -p services/${SERVICE_NAME}

RUN cat <<EOF > package.json
{
    "name": "project-bundle",
    "type": "module",
    "dependencies": {
        "@project/${PACKAGE_NAME}": "file:packages/${PACKAGE_NAME}"
    }
}
EOF

RUN cp -r /opt/build/packages/${PACKAGE_NAME}/dist ./packages/${PACKAGE_NAME}/dist
RUN cp /opt/build/packages/${PACKAGE_NAME}/package.json ./packages/${PACKAGE_NAME}/

RUN cp -r /opt/build/services/${SERVICE_NAME}/dist ./services/${SERVICE_NAME}/dist
RUN cp /opt/build/services/${SERVICE_NAME}/package.json ./services/${SERVICE_NAME}/

RUN npm install

# running container
FROM node:22-alpine

WORKDIR /usr/app

COPY --from=0 /opt/app ./

CMD ["node", "--enable-source-maps", "services/${SERVICE_NAME}/dist/index.js"]
