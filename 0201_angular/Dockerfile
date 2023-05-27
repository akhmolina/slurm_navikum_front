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

ENV CHROMIUM_FLAGS="--no-sandbox"
ENV CHROME_BIN=/usr/bin/google-chrome

RUN apt-get update && apt-get install -y wget
RUN wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
RUN apt-get install -y ./google-chrome-stable_current_amd64.deb

RUN npm run test | tee testreport

# lint
FROM source as lint

RUN npm run lint | tee lintresult

# run
FROM nginx:1.22 as final

COPY --from=lint /ng-app/lintresult /ng-app/lintresult
COPY --from=test /ng-app/testreport /ng-app/testreport
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder --chown=nginx:nginx /ng-app/storybook-static /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/

CMD nginx -g 'daemon off;'