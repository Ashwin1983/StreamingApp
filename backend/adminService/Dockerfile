FROM node:18-alpine

WORKDIR /app

COPY adminService/package*.json ./

RUN npm ci --omit=dev

COPY adminService/. ./

ENV NODE_ENV=production

EXPOSE 3003

CMD ["npm", "start"]
