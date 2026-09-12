FROM node:18-alpine

WORKDIR /app

COPY backend/streamingService/package*.json ./

RUN npm ci --omit=dev

COPY backend/streamingService/. ./

ENV NODE_ENV=production

EXPOSE 3002

CMD ["npm", "start"]
