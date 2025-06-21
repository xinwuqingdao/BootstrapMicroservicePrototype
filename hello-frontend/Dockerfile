# Stage 1: Build
FROM node:22 as build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build --prod

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist/hello-frontend/browser /usr/share/nginx/html
EXPOSE 80
