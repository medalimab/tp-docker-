FROM node:lts-alpine

# Create app directory
WORKDIR /app

# Install app dependencies
COPY package*.json ./
RUN npm install --production

# Copy app source
COPY . .

ENV PORT=9000
EXPOSE 9000

# Start the server
CMD ["npm", "start"]
