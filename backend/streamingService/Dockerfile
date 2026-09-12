FROM node:18-alpine

WORKDIR /app

COPY streamingService/package*.json ./

RUN npm ci --omit=dev

COPY streamingService/. ./

ENV NODE_ENV=production

EXPOSE 3002

CMD ["npm", "start"]
