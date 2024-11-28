# Sample Repository from the Internet and Applying DevOps Tooling

### 1. **Repository Overview**

I have selected this Node.js web application [repository](https://github.com/expressjs/express). It's a basic web framework used to build applications and APIs, containing `package.json` for dependencies and a simple server script.

### 2. **Applying DevOps Tools**

#### **Docker**

*   **Containerization**: Add a `Dockerfile` to define the container build process.

    **Dockerfile**:
    ```dockerfile
    FROM node:16
    WORKDIR /usr/src/app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 3000
    CMD ["node", "app.js"]
    ```

*   **Build & Run Commands**:
    ```bash
    # Build the Docker image
    docker build -t express-app .

    # Run the container
    docker run -p 3000:3000 express-app
    ```

#### **Kubernetes**

*   **Set Up Kubernetes Cluster**: Use a tool like `minikube` or a cloud-based service (e.g., AWS EKS) to set up a Kubernetes cluster.

*   **Deployment Configuration**: Create `deployment.yaml` for deployment and service configuration.

    **deployment.yaml**:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: express-app
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: express-app
      template:
        metadata:
          labels:
            app: express-app
        spec:
          containers:
          - name: express-app
            image: express-app:latest
            ports:
            - containerPort: 3000
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: express-app-service
    spec:
      type: LoadBalancer
      ports:
      - port: 80
        targetPort: 3000
      selector:
        app: express-app
    ```

#### **Jenkins**

*   **Setup CI/CD Pipeline**:
    - **Install Jenkins**: Deploy Jenkins via Docker or a cloud service (e.g., Jenkins on AWS).
    - **Pipeline Configuration**: Create a `Jenkinsfile`.

    **Jenkinsfile**:
    ```groovy
    pipeline {
        agent any
        stages {
            stage('Checkout Code') {
                steps {
                    git 'https://github.com/expressjs/express.git'
                }
            }
            stage('Build Docker Image') {
                steps {
                    sh 'docker build -t express-app .'
                }
            }
            stage('Run Tests') {
                steps {
                    sh 'npm test'
                }
            }
            stage('Push to Docker Hub') {
                steps {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                        sh 'docker tag express-app username/express-app:latest'
                        sh 'docker push username/express-app:latest'
                    }
                }
            }
            stage('Deploy to Kubernetes') {
                steps {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
    ```

#### **GitHub Actions**

*   **Set Up CI/CD Workflow**: Create a `.github/workflows/main.yml` file for automated CI/CD.

    **.github/workflows/main.yml**:
    ```yaml
    name: CI/CD Pipeline

    on:
      push:
        branches:
          - main

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
        - name: Checkout Code
          uses: actions/checkout@v3
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '16'
        - name: Install Dependencies
          run: npm install
        - name: Run Tests
          run: npm test
        - name: Build Docker Image
          run: docker build -t username/express-app .
        - name: Push Docker Image
          env:
            DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
            DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
          run: |
            echo "${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}" | docker login -u "${{ secrets.DOCKER_HUB_USERNAME }}" --password-stdin
            docker tag express-app username/express-app:latest
            docker push username/express-app:latest
      deploy:
        runs-on: ubuntu-latest
        steps:
        - name: Set up Kubernetes CLI
          uses: azure/setup-kubectl@v3
          with:
            version: 'latest'
        - name: Deploy to Kubernetes
          run: kubectl apply -f deployment.yaml
    ```

### 3. **Summary**

Applying Docker, Kubernetes, Jenkins, and GitHub Actions to this repository creates a robust CI/CD pipeline that automates code integration, testing, containerization, and deployment. This process ensures efficient, reliable, and repeatable software delivery, enhancing development productivity and reducing manual intervention.
