### STAGE 1: Build ###
FROM node:16.16.0-alpine AS build
WORKDIR /usr/src/app
COPY package.json ./
RUN npm install --legacy-peer-deps --location=global npm@8.16.0 \
  && npm install --legacy-peer-deps --location=global @angular/cli \
  && npm install --legacy-peer-deps --save-dev @angular-devkit/build-angular
COPY . .
RUN npm run build
### STAGE 2: Run ###
FROM nginx:1.23.1-alpine
COPY --from=build /usr/src/app/dist/hello-spa /usr/share/nginx/html
