FROM node:24.7.0-alpine3.22

COPY src /app/
COPY package.json /app/

WORKDIR /app

RUN npm install --production

EXPOSE 3030

CMD ["node", "index.js"]
