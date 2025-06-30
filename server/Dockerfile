# Base image for Node.js
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy server files
COPY . .

# Expose server port
EXPOSE 5000

# Start the server
CMD ["node", "server.js"]
