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

ENV CHROMIUM_FLAGS="--no-sandbox --headless --disable-setuid-sandbox --disable-gpu --disable-software-rasterizer --disable-dev-shm-usage --disable-audio-output --dbus-stub --single-process"
ENV CHROME_BIN=/usr/bin/chromium
ENV CHROME_PATH=/usr/lib/chromium/
ENV CI=true

RUN apt-get update && apt-get install -y xvfb dbus chromium
RUN mkdir -p /var/run/dbus && \
    dbus-daemon --config-file=/usr/share/dbus-1/system.conf --print-address

RUN xvfb-run npm run test
RUN touch .test

# lint
FROM source as lint

RUN npm run lint
RUN touch .lint

# run
FROM nginx:1.22 as final

COPY --from=lint /ng-app/.lint /ng-app/.lint
COPY --from=test /ng-app/.test /ng-app/.test
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder --chown=nginx:nginx /ng-app/storybook-static /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/