FROM node:16 as build
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
RUN npm install -g @angular/cli@14.2.1
COPY . .
RUN ng build --configuration production
FROM nginx:alpine
COPY --from=build /usr/src/app/dist/angular-frontend /usr/share/nginx/html
EXPOSE 80
