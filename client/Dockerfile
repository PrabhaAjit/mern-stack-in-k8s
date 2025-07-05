# Stage 1: build React app
FROM node:18-alpine AS build
WORKDIR /app

# Accept build argument
#ARG REACT_APP_YOUR_HOSTNAME
# Set it as env var for the build
#ENV REACT_APP_YOUR_HOSTNAME=$REACT_APP_YOUR_HOSTNAME

COPY package*.json ./
RUN npm install
COPY . .
# Build with that env var baked in
RUN npm run build

# Stage 2: serve with Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
