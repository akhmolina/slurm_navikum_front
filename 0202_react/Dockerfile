# copy source code
FROM node:18-alpine as builder

RUN mkdir /ng-app
WORKDIR /ng-app

COPY package.json yarn.lock ./
RUN yarn install --immutable

COPY . .
RUN yarn run build

# run
FROM nginx:1.22-alpine as final

RUN apk upgrade --update-cache --available && \
    apk add openssl && \
    rm -rf /var/cache/apk/*
RUN mkdir /etc/nginx/keys/ && \
    openssl req -x509 -nodes -days 30 -subj "/C=CA/ST=QC/O=Company, Inc./CN=localhost" \
    -addext "subjectAltName=DNS:localhost" \ 
    -newkey rsa:4096 -keyout /etc/nginx/keys/server.key -out /etc/nginx/keys/server.crt && \
    chown nginx:nginx /etc/nginx/keys

RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder --chown=nginx:nginx /ng-app/build /usr/share/nginx/html
COPY nginx/default.conf.template /etc/nginx/templates/default.conf.template

EXPOSE 8080:8443
CMD nginx -g 'daemon off;'