# Use official Node.js image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy all app files
COPY . .

# Build the React app for production
RUN npm run build

# Install a simple static file server to serve the build output
RUN npm install -g serve

# Expose the port that 'serve' will use
EXPOSE 3000

# Command to serve the React app
CMD ["serve", "-s", "build", "-l", "3000"]

