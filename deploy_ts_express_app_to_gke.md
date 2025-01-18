
# Deploying a TypeScript Express App to GKE Using Docker and Artifact Registry

This guide walks you through the steps to create a TypeScript Express app, containerize it with Docker, and deploy it to Google Kubernetes Engine (GKE) using Google Artifact Registry.

---

## **1. Create a TypeScript Express App**
1. **Initialize the Project**:
   ```bash
   mkdir ts_express_server
   cd ts_express_server
   npm init -y
   ```

2. **Install Required Packages**:
   ```bash
   npm install express cors
   npm install --save-dev typescript @types/node @types/express @types/cors
   ```

3. **Set Up TypeScript**:
   Create a `tsconfig.json` file:
   ```json
   {
     "compilerOptions": {
       "outDir": "./dist",
       "module": "commonjs",
       "target": "es6",
       "strict": true
     },
     "include": ["src/**/*"]
   }
   ```

4. **Write the Server Code**:
   Create `src/server.ts`:
   ```typescript
   import express, { Request, Response } from "express";
   import Cors from "cors";

   const app = express();
   app.use(express.json());
   app.use(Cors());

   app.get("/api", (req: Request, res: Response) => {
     res.status(200).json({ message: "Server up " + new Date().toISOString() });
   });

   app.listen(8090, () => {
     console.log("Server listening on port 8090");
   });
   ```

5. **Build the App**:
   Add build scripts to `package.json`:
   ```json
   "scripts": {
     "build": "tsc",
     "start": "node dist/server.js"
   }
   ```

   Build the app:
   ```bash
   npm run build
   ```

---

## **2. Create a Docker Image**
1. **Write a `Dockerfile`**:
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build
   EXPOSE 8090
   CMD ["npm", "run", "start"]
   ```

2. **Build the Docker Image**:
   ```bash
   docker build -t emirob/emi-repo:my-express-app .
   ```

3. **Push the Image to Docker Hub**:
   ```bash
   docker tag emirob/emi-repo:my-express-app emirob/emi-repo:my-express-app
   docker push emirob/emi-repo:my-express-app
   ```

---

## **3. Set Up GCP for Artifact Registry and GKE**
1. **Enable Required APIs**:
   - Go to the Google Cloud Console.
   - Enable the **Artifact Registry API**.
   - Enable the **Kubernetes Engine API**.

2. **Set Your Project in Cloud Shell**:
   ```bash
   export PROJECT_ID=gketest-448214
   gcloud config set project $PROJECT_ID
   ```

3. **Create an Artifact Registry Repository**:
   ```bash
   gcloud artifacts repositories create emi-repo        --repository-format=docker        --location=us-west1        --description="Docker repo"
   ```

4. **Authenticate Docker with GCP**:
   ```bash
   gcloud auth configure-docker us-west1-docker.pkg.dev
   ```

---

## **4. Push the Docker Image to Artifact Registry**
1. **Pull the Image from Docker Hub**:
   ```bash
   docker pull emirob/emi-repo:my-express-app
   ```

2. **Tag the Image for Artifact Registry**:
   ```bash
   docker tag emirob/emi-repo:my-express-app        us-west1-docker.pkg.dev/gketest-448214/emi-repo/my-express-app:latest
   ```

3. **Push the Image to Artifact Registry**:
   ```bash
   docker push us-west1-docker.pkg.dev/gketest-448214/emi-repo/my-express-app:latest
   ```

4. **Verify the Image**:
   ```bash
   gcloud artifacts docker images list us-west1-docker.pkg.dev/gketest-448214/emi-repo
   ```

---

## **5. Deploy the App on GKE**
1. **Create a GKE Cluster**:
   ```bash
   gcloud container clusters create my-cluster        --num-nodes=3        --zone=us-west1-a
   ```

2. **Connect to the Cluster**:
   ```bash
   gcloud container clusters get-credentials my-cluster --zone us-west1-a
   ```

3. **Create a Kubernetes Deployment**:
   Save the following as `deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-express-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-express-app
     template:
       metadata:
         labels:
           app: my-express-app
       spec:
         containers:
         - name: my-express-app
           image: us-west1-docker.pkg.dev/gketest-448214/emi-repo/my-express-app:latest
           ports:
           - containerPort: 8090
   ```

4. **Apply the Deployment**:
   ```bash
   kubectl apply -f deployment.yaml
   ```

5. **Expose the Service**:
   ```bash
   kubectl expose deployment my-express-app --type=LoadBalancer --port 80 --target-port 8090
   ```

6. **Access the Application**:
   Get the external IP of the LoadBalancer:
   ```bash
   kubectl get service my-express-app
   ```
   Visit `http://<EXTERNAL-IP>/api`.
---
## **6. GitHub Repository**
For the full project setup and Docker image creation, check the GitHub repository:  
[build_node_server_docker_images](https://github.com/EmiRoberti77/build_node_server_docker_images)

Emi Roberti - Happy coding
