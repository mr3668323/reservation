# Use an official Node.js runtime to build the frontend
FROM node:18-alpine as builder

# Set the working directory in the container
WORKDIR /app/frontend

# Copy package.json and package-lock.json
COPY ./package*.json ./

# Install frontend dependencies
RUN npm install

# Copy the rest of the frontend application code
COPY . .

# Build the frontend application (adjust the build command if needed)
RUN npm run build

# Use a lightweight web server (nginx) to serve the static build
FROM nginx:alpine

# Copy the built frontend assets from the builder stage
COPY --from=builder /app/frontend/dist /usr/share/nginx/html

# Expose port 80 for the web server
EXPOSE 80

# Command to start nginx
CMD ["nginx", "-g", "daemon off;"]
