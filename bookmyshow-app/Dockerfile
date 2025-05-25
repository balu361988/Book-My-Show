FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package definition and lock files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy all application source code
COPY . .

# Expose the app port
EXPOSE 3000

# Start the app
CMD ["npm", "start"]

