FROM node:14-alpine

env NODE_ENV=production

WORKDIR /app

COPY package*.json .

RUN npm ci

COPY . .

CMD ["npm", "start"]