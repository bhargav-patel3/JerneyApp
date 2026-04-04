FROM node:18-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --0nly=production && npm cache clean --force

FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
