FROM node:20-alpine
WORKDIR /app

# dependency layer
COPY package.json yarn.lock ./
RUN yarn install --production

# work layer
COPY . .
EXPOSE 3000

# startup
CMD ["node", "src/index.js"]
