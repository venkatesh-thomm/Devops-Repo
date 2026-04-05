FROM node:18-alpine
WORKDIR /app

# Install dependencies first to cache
COPY package*.json ./
RUN npm install

# Copy app source
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]