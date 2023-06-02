# copy source code
FROM node:18-buster-slim as source

RUN mkdir /ng-app
WORKDIR /ng-app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

# build
FROM source as builder

RUN npm run build-storybook

# test
FROM source as test

ENV CHROMIUM_FLAGS="--no-sandbox --headless --remote-debugging-port=9222"
ENV CHROME_BIN=/usr/bin/chromium
ENV CHROME_PATH=/usr/lib/chromium/
ENV CI=true

RUN apt-get update && apt-get install --no-install-recommends -y chromium

RUN npm run test --watch=false
RUN touch .test

# lint
FROM source as lint

RUN npm run lint
RUN touch .lint

# run
FROM nginx:1.22 as final

RUN rm -rf /usr/share/nginx/html/*
COPY nginx/default.conf /etc/nginx/conf.d/
COPY --from=builder --chown=nginx:nginx /ng-app/storybook-static /usr/share/nginx/html
COPY --from=lint /ng-app/.lint /ng-app/.lint
COPY --from=test /ng-app/.test /ng-app/.test