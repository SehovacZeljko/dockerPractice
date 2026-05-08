FROM node:14.16.0-alpine3.13

RUN addgroup app && adduser -S -G app app
USER app

WORKDIR /app
COPY --chown=app:app package*.json ./
RUN npm install
COPY --chown=app:app . .
RUN chmod +x docker-entrypoint.sh wait-for

EXPOSE 3001

CMD ["npm", "start"]