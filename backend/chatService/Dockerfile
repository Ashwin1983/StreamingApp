FROM node:18-alpine

WORKDIR /app

COPY chatService/package*.json ./

RUN npm ci --omit=dev

COPY chatService/. ./

ENV NODE_ENV=production

EXPOSE 3004

CMD ["npm", "start"]
