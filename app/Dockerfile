# Selecting the base image to build our own customised node.js application microservice
FROM node:alpine

# Working directory inside the container
WORKDIR /usr/src/app

# Copying dependencies
COPY package*.json ./


# Setting env variables
#ARG DB_HOST="mongodb://localhost:27017/posts"
#ENV DB_HOST=${DB_HOST}

# Exporting DB
#RUN export DB_HOST="mongodb://localhost:27017/posts"


# Installing node package manager
RUN npm install

# Copying everything from current location to default location inside the container
COPY . .

# Expose the port
EXPOSE 3000

# Starting the app with CMD - 
CMD ["node", "app.js"]


