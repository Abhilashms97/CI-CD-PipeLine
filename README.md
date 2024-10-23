**# CI-CD-PipeLine & Docker**

1. Web App Deployment: CI/CD Pipeline Setup
Objective: Automate the process of deploying your web app (abhi.icamp.tech) to your server whenever there are updates to the codebase, specifically when changes are pushed to the main branch.
Step 1.1: Setup GitHub Actions (CI/CD Pipeline)
    • Purpose: To automate tasks such as code testing, building, and deployment.
    • Actions:
        1. Create a GitHub Workflow File (main.yml):
            ▪ This file defines the actions to take whenever code is pushed or a pull request is made on the main branch.
        2. Check Out Code:
            ▪ This action uses actions/checkout@v4.2.1 to pull the code from your GitHub repository.
        3. Set Up Node.js:
            ▪ We set up Node.js (version 20) using actions/setup-node@v4.0.4 to ensure that the environment is ready for building and running your JavaScript-based app.
        4. Install Dependencies:
            ▪ Using npm install, all the necessary dependencies for the project are installed based on the package.json file.
        5. Build the App:
            ▪ If your project has a build step (e.g., npm run build), the app is compiled into a production-ready state.
        6. Deploy to the Server:
            
▪ We used appleboy/scp-action@v0.1.7 to deploy the app from GitHub to the server. This action uses SSH to securely copy files to the specified server path /var/www/app name.
    • Key YML file for the GitHub Actions Workflow:

yaml

name: CI/CD Pipeline
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4.2.1

    - name: Set up Node.js
      uses: actions/setup-node@v4.0.4
      with:
        node-version: '20'

    - name: Install dependencies
      run: npm install

    - name: Build the app
      run: npm run build

    - name: Deploy to server
      uses: appleboy/scp-action@v0.1.7
      with:
        host: ip address
        username: root
        key: ${{ secrets.SSH_KEY }}
        source: "./*"
        target: "/var/www/app name"

Step 1.2: Set Up Server (Basic Configuration)
    • Objective: Ensure the server is prepared to receive files from the CI/CD pipeline.
        ◦ You needed to configure your SSH key for secure communication between GitHub Actions and your server.
        ◦ Server Path: /var/www/app name
        ◦ Server IP: 0.0.0.0
Result: Once this setup was complete, you had a fully automated system for deploying your app every time you pushed to the main branch.

*2. Dockerizing the Application*

Objective: To encapsulate your app in a Docker container, allowing it to run consistently across different environments (development, staging, production, etc.).
Step 2.1: Install Docker
    • Action: You installed Docker on your server to allow containerization.
      
      sudo apt install docker.io

Step 2.2: Create Dockerfile
    • Purpose: A Dockerfile is a text file that contains all the instructions needed to build your app into a Docker image.
    • Dockerfile Breakdown:
        ◦ Base Image:
            ▪ Uses node:20-alpine as the base image, which is a lightweight version of Node.js 20 built on Alpine Linux.
        ◦ Working Directory:
            ▪ Sets the working directory in the container to /app.
        ◦ Copy Files and Install Dependencies:
            ▪ Copies package.json files to the container and runs npm install to install dependencies.
        ◦ Copy the Entire App:
            ▪ Copies all the application files from the host to the container.
        ◦ Expose the App’s Port:
            ▪ Exposes port 3000, which is the port the app listens to inside the container.
        ◦ Start the App:
            ▪ Runs the app using the npm start command.
    
• Your Dockerfile:

dockerfile
# Base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application
COPY . .

# Expose port 3000
EXPOSE 3000

# Command to start the app
CMD ["npm", "start"]

Step 2.3: Build Docker Image
    • Command:
    
      docker build -t app name .
    • Explanation:
        ◦ docker build creates a Docker image using the Dockerfile.
        ◦ -t app name assigns the tag to the image for easy identification.

Step 2.4: Run Docker Container
    • Command:
      
      docker run -p 3001:3000 abhi.icamp.tech
    • Explanation:
        ◦ This command starts a new Docker container from the abhi.icamp.tech image.
        ◦ -p 3001:3000 maps port 3000 inside the container to port 3001 on the host machine (your server).

Result: Your app was successfully containerized and could be accessed via http://localhost:3001 (on your server) or its IP address externally.

3. Docker and CI/CD Integration
Objective: Automate Docker image building and container deployment using the CI/CD pipeline.
Step 3.1: Modify CI/CD Pipeline for Docker
    • In addition to code build and deployment, you wanted to integrate Docker into the pipeline, automating the build and running of Docker containers after each push to main.
Pipeline Steps:
    1. Checkout Code: Pull the latest code from the repository.
    2. Set Up Node.js: Ensures the project’s Node.js environment is set up.
    3. Install Dependencies: Installs all necessary dependencies for the project using npm install.
    4. Build Docker Image:
        ◦ Builds the Docker image using the Dockerfile in the repository.
       
       docker build -t appname .

    5. Stop Old Docker Containers:
        ◦ This command stops any existing containers running from the previous build.
       
       
       docker stop $(docker ps -q --filter "ancestor=appname") || true

    6. Run the Docker Container:
        ◦ Runs the newly built Docker image as a container and maps the necessary ports.
      
       docker run -d -p 3001:3000 appname

   
 • Updated CI/CD Workflow with Docker Integration:
yaml

name: CI/CD Pipeline with Docker

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4.2.1

    - name: Set up Node.js
      uses: actions/setup-node@v4.0.4
      with:
        node-version: '20'

    - name: Install dependencies
      run: npm install

    - name: Build the Docker image
      run: docker build -t appname .

    - name: Deploy Docker container
      run: |
        docker stop $(docker ps -q --filter "ancestor=appname") || true
        docker run -d -p 3001:3000 app name

Result: Now, whenever you push to the main branch, GitHub Actions will:
    • Build a new Docker image
    • Stop any running containers from the previous build
    • Deploy a new Docker container on your server.

4. Accessing and Managing Docker Containers
Once the Docker setup was complete, you learned how to interact with and manage Docker containers:
Access Docker Containers:
    • Command:
     
      docker ps -a

    • Explanation: This command lists all running and stopped containers.
Stop a Running Container:
    • Command:
     
      docker stop [container_id]

    • Explanation: Stops the running Docker container identified by its container_id.
View Docker Logs:
    • Command:
     
      docker logs [container_id]

    • Explanation: Displays logs generated by the container during its execution.
Remove a Docker Container:
    • Command:

      docker rm [container_id]

    • Explanation: Permanently removes a stopped container.
