FROM node:20-alpine

WORKDIR /app
COPY package.json .
RUN npm install
RUN apk add curl
COPY src ./src

CMD ["node", "src/index.js"]