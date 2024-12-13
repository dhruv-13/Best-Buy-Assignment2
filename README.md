# Assignment #2: Building a Cloud-Native App for Best Buy

## Objective

The assignment is focused on implementing a microservices-based application that incorporates AI capabilities and is deployed in a Kubernetes cluster. The app is inspired by the Algonquin Pet Store architecture but introduces a critical modification: the use of a managed order queue service (e.g., Azure Service Bus) instead of RabbitMQ.

## Key Highlights:
1. Microservices Architecture: Each service is independently developed and deployed, ensuring scalability and fault isolation.
2. AI-Powered Features: Leverage GPT-4 and DALL-E for generating product descriptions and images.
3. Cloud Deployment: Deploy the application using Kubernetes, ensuring high availability and efficient resource management.

## Application Overview

The application consists of the following components:


| Service          | Description                                     | Key Notes                                   |
|-------------------|-------------------------------------------------|---------------------------------------------|
| **Store-Front**   | Customer-facing app for browsing and placing orders. | User-friendly interface for customers.      |
| **Store-Admin**   | Employee-facing app for managing products and viewing orders. | Operational tools for employees.           |
| **Order-Service** | Handles order creation and sends data to the managed order queue. | Replaces RabbitMQ with Azure Service Bus.  |
| **Product-Service**| Handles CRUD operations for product data.      | Backbone for product information.           |
| **Makeline-Service**| Processes and completes orders by reading from the order queue. | Finalizes order preparation.               |
| **AI-Service**    | Generates product descriptions and images.      | Leverages GPT-4 and DALL-E models.          |
| **Database**      | MongoDB for persisting order and product data.  | Reliable and scalable data storage.         |

## Architecture 

![Untitled diagram-2024-12-13-015913](https://github.com/user-attachments/assets/b68abdd9-185a-4b6c-a783-17f113bbc0a3)

## Architecture Flow Explanation
This cloud-native application is designed to operate seamlessly within a Kubernetes-based infrastructure (AKS Cluster) while leveraging external services like Azure OpenAI for AI-powered functionalities. 

| **Layer**                   | **Components**                                                                                               | **Description**                                                                                                                                                       |
|-----------------------------|------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Interaction Layer**        | **Customers**, **Admin**, **Store-Front (Vue.js)**, **Store-Admin (Vue.js)**                              | - Customers browse products and place orders via the **Store-Front** app. <br> - Admin manages products and orders via the **Store-Admin** app. Both are deployed in the AKS Cluster. |
| **Business Logic Layer**     | **Order-Service (Node.js)**, **Product-Service (Rust)**, **Makeline-Service (Go)**                       | - **Order-Service**: Handles order creation, stores data in MongoDB, and sends messages to RabbitMQ.<br> - **Product-Service**: Manages product CRUD operations, integrates AI-enhanced data.<br> - **Makeline-Service**: Consumes RabbitMQ messages to process and complete orders. |
| **Message Queue Layer**      | **RabbitMQ**                                                                                            | - Acts as the message broker for decoupling **Order-Service** and **Makeline-Service**.<br> - Enables asynchronous communication and smooth order processing.      |
| **Data Layer**               | **MongoDB Database**                                                                                     | - Stores persistent data for both orders and products.<br> - Acts as a unified data source for backend services.                                                   |
| **AI-Powered Enhancements**  | **AI-Service (Python)**, **Azure OpenAI Service (GPT-4, DALL-E)**                                        | - **Azure OpenAI Service**: Provides AI models like GPT-4 and DALL-E for product descriptions and images.<br> - **AI-Service**: Integrates AI-generated data with product services. |
| **Deployment Infrastructure**| **AKS Cluster**                                                                                         | - Hosts all microservices (front-end, backend, and AI components).<br> - Ensures scalability, reliability, and fault tolerance.<br> - Deployed in a Kubernetes-based environment. |

## Workflow

| **Step**                     | **Flow**                                                                                                 | **Description**                                                                                                                                                       |
|-----------------------------|--------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1. Customer Browses**       | Customers → **Store-Front (Vue.js)**                                                                  | Customer browses products using the **Store-Front** app.                                                                                                           |
| **2. Order Creation**         | **Store-Front** → **Order-Service (Node.js)**                                                         | Order is sent to the **Order-Service**, stored in MongoDB, and pushed to RabbitMQ.                                                                                 |
| **3. Order Processing**       | **RabbitMQ** → **Makeline-Service (Go)**                                                              | The **Makeline-Service** processes the order, finalizes the workflow, and updates the system.                                                                       |
| **4. AI-Enhanced Products**   | **Product-Service (Rust)** → **AI-Service (Python)** → **Azure OpenAI**                               | The **Product-Service** requests AI-generated product descriptions (via GPT-4) and images (via DALL-E) from the **AI-Service**, which communicates with **Azure OpenAI**. |
| **5. Admin Interaction**      | Admin → **Store-Admin (Vue.js)**                                                                      | Admin uses the **Store-Admin** app to manage product listings, view enhanced product data, and track orders.                                                       |

## Deploying the Application

### Step 1: Install `kubectl`

1. **What is `kubectl`?**
   - `kubectl` is a command-line tool that allows you to communicate with and manage Kubernetes clusters. You will use `kubectl` to deploy applications, configure clusters, and inspect resources.

2. **Installing `kubectl`:**
   - Follow the official installation guide to install `kubectl` on your system. The guide provides detailed instructions for various operating systems.
   - [kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/)

3. **Verify `kubectl` Installation:**
   - After installing, confirm that `kubectl` is properly set up by running:
     ```bash
     kubectl version --client
     ```
   - You should see the client version information displayed, confirming a successful installation.

### Step 2: Create an Azure Kubernetes Cluster (AKS)

1. **Log in to Azure Portal:**
   - Go to [https://portal.azure.com](https://portal.azure.com) and log in with your Azure account.

2. **Create a Resource Group:**
   - In the Azure Portal, search for **Resource Groups** in the search bar.
   - Click **Create** and fill in the following:
     - **Resource group name**: `BestbuyApp`
     - **Region**: `Canada-Central`.
   - Click **Review + Create** and then **Create**.

3. **Create an AKS Cluster:**
   - In the search bar, type **Kubernetes services** and click on it.
   - Click **Create** and select **Kubernetes cluster**
   - In the `Basics` tap fill in the following details:
     - **Subscription**: Select your subscription.
     - **Resource group**: Choose `BestbuyApp`.
     - **Cluster preset configuration**: Choose `Dev/Test`.
     - **Kubernetes cluster name**: `BestbuyCluster`.
     - **Region**: Same as your resource group (e.g., `Canada`).
     - **Availability zones**: `None`.
     - **AKS pricing tier**: `Free`.
     - **Kubernetes version**: `Default`.
     - **Automatic upgrade**: `Disabled`.
     - **Automatic upgrade scheduler**: `No schedule`.
     - **Node security channel type**: `None`.
     - **Security channel scheduler**: `No schedule`.
     - **Authentication and Authorization**: `Local accounts with Kubernetes RBAC`.
   - In the `Node pools` tap fill in the following details:
     - Select **agentpool**. Optionally change its name to `masterpool`. This nodes will have the controlplane.
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `1`
        - Click `update`
     - Click on **Add node pool**:
        - **Node pool name**: `workerspool`.
        - **Mode**: `User` 
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `2`
        - Click `add`
   - Click **Review + Create**, and then **Create**. The deployment will take a few minutes.
  
  4. **Connect to the AKS Cluster:**
   - Once the AKS cluster is deployed, navigate to the cluster in the Azure Portal.
   - In the overview page, click on **Connect**. 
   - Select **Azure CLI** tap. You will need Azure CLI. If you don't have it: [**Install Azure CLI**](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
   - Login to your azure account using the following command:
      ```
      az login
      ```
   - Set the cluster subscription using the command shown in the portal (it will look something like this):
      ```
      az account set --subscription 'subscribtion-id'
      ```

   - Copy the command shown in the portal for configuring `kubectl` (it will look something like this):
     ```
     az aks get-credentials --resource-group BestbuyApp --name BestbuyCluster
     ```
      **Understanding the Command:**
      - The command `az aks get-credentials` pulls the necessary configuration files to enable `kubectl` to access your AKS cluster. Here’s a breakdown:
     - `--resource-group` specifies the resource group where your AKS cluster resides.
     - `--name` specifies the name of your AKS cluster.
     - `--overwrite-existing` can be used to overwrite any existing Kubernetes configuration files for the same cluster. This is useful if you’ve connected to the cluster before or if multiple configurations exist for it.
   - Verify Cluster Access:
      - Test your connection to the AKS cluster by listing all nodes:
        ```
        kubectl get nodes
        ```
        You should see details of the nodes in your AKS cluster if the connection is successful.
---

### Step 3: Set Up the AI Backing Services
To enable AI-generated product descriptions and image generation features, you will deploy the required **Azure OpenAI Services** for GPT-4 (text generation) and DALL-E 3 (image generation). This step is essential to configure the **AI Service** component in the Algonquin Pet Store application.

#### Task 1: Create an Azure OpenAI Service Instance

1. **Navigate to Azure Portal**:
   - Go to the [Azure Portal](https://portal.azure.com/).

2. **Create a Resource**:
   - Select **Create a Resource** from the Azure portal dashboard.
   - Search for **Azure OpenAI** in the marketplace.

3. **Set Up the Azure OpenAI Resource**:
   - Choose the **East US** region for deployment to ensure capacity for GPT-4 and DALL-E 3 models.
   - Fill in the required details:
     - Resource group: Use an existing one or create a new group.
     - Pricing tier: Select `Standard`.

4. **Deploy the Resource**:
   - Click **Review + Create** and then **Create** to deploy the Azure OpenAI service.

---

#### Task 2: Deploy the GPT-4 and DALL-E 3 Models

1. **Access the Azure OpenAI Resource**:
   - Navigate to the Azure OpenAI resource you just created.

2. **Deploy GPT-4**:
   - Go to the **Model Deployments** section and click **Add Deployment**.
   - Choose **GPT-4** as the model and provide a deployment name (e.g., `gpt-4-deployment`).
   - Set the deployment configuration as required and deploy the model.

3. **Deploy DALL-E 3**:
   - Repeat the same process to deploy **DALL-E 3**.
   - Use a descriptive deployment name (e.g., `dalle-3-deployment`).

4. **Note Configuration Details**:
   - Once deployed, note down the following details for each model:
     - Deployment Name
     - Endpoint URL
---

#### Task 3: Retrieve and Configure API Keys

1. **Get API Keys**:
   - Go to the **Keys and Endpoints** section of your Azure OpenAI resource.
   - Copy the **API Key (API key 1)** and **Endpoint URL**.

2. **Base64 Encode the API Key**:
   - Use the following command to Base64 encode your API key:
     ```bash
     echo -n "<your-api-key>" | base64
     ```
   - Replace `<your-api-key>` with your actual API key.

---
### Task 4: Update AI Service Deployment Configuration in the `Deployment Files` folder.
1. **Modify Secretes YAML**:
   - Edit the `secrets.yaml` file.
   - Replace `OPENAI_API_KEY` placeholder with the Base64-encoded value of the `API_KEY`. 
2. **Modify Deployment YAML**:
   - Edit the `aps-all-in-one.yaml` file.
   - Replace the placeholders with the configurations you retrieved:
     - `AZURE_OPENAI_DEPLOYMENT_NAME`: Enter the deployment name for GPT-4.
     - `AZURE_OPENAI_ENDPOINT`: Enter the endpoint URL for the GPT-4 deployment.
     - `AZURE_OPENAI_DALLE_ENDPOINT`: Enter the endpoint URL for the DALL-E 3 deployment.
     - `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME`: Enter the deployment name for DALL-E 3.

   Example configuration in the YAML file:
   ```yaml
   - name: AZURE_OPENAI_API_VERSION
     value: "2024-07-01-preview"
   - name: AZURE_OPENAI_DEPLOYMENT_NAME
     value: "gpt-4"
   - name: AZURE_OPENAI_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_DEPLOYMENT_NAME
     value: "dall-e-3"
   ```
## Step 4: Deploy the ConfigMaps and Secrets
- Deploy the ConfigMap for RabbitMQ Plugins:
   ```bash
   kubectl apply -f config-maps.yaml
   ```
- Create and Deploy the Secret for OpenAI API:  
   - Make sure that you have replaced Base64-encoded-API-KEY in secrets.yaml with your Base64-encoded OpenAI API key.
   ```bash
   kubectl apply -f secrets.yaml
   ```
- Verify:
   ```bash
   kubectl get configmaps
   kubectl get secrets
   ```

## Step 5: Deploy the Application
   ```bash
   kubectl apply -f aps-all-in-one.yaml
   ```
### Validate the Deployment
- Check Pods and Services:
   ```bash
   kubectl get pods
   kubectl get services
   ```
- Test Frontend Access:
   - Locate the external IPs for store-front and store-admin services:
   ```bash
   kubectl get services
   ```
   - Access the Store Front app at the external IP on port 80.
   - Access the Store Admin app at the external IP on port 80.
## Step 6: Deploy Virtual Customer and Worker
   ```bash
   kubectl apply -f admin-tasks.yaml
   ```
- Monitor Virtual Customer:
   ```bash
   kubectl logs -f deployment/virtual-customer
   ```
- Monitor Virtual Worker:
   ```bash
   kubectl logs -f deployment/virtual-worker
   ```

## Step 7: Scale and Monitor Services
### Scale Deployments:
- Scale the `order-service` to 3 replicas:
```bash
kubectl scale deployment order-service --replicas=3
```
- Check Scaling:
```bash
kubectl get pods
```
- Monitor Resource Usage:

   - Enable metrics server for resource monitoring.
   - Use kubectl top to monitor pod and node usage:
   ```bash
   kubectl top pods
   kubectl top nodes
   ```

## Step 8: Explore Advanced Features
### AI-Generated Descriptions and Images:
- Use the AI Service for generating product descriptions and images.
- Ensure your OpenAI API key is correctly configured in the deployed secret.
### RabbitMQ Management:
- Access the RabbitMQ management UI:
   ```bash
   kubectl port-forward service/rabbitmq 15672:15672
   ```
   The kubectl port-forward command is used to forward a local port to a port on a Kubernetes resource (e.g., a Pod or Service). This allows you to access the application running in the cluster from your local machine without exposing it externally.


- Login with the default credentials (`username`/`password`).

### MongoDB Shell Access and Database Exploration
In this section, you will use the MongoDB shell to interact with the `orderdb` database, which stores order information for the Algonquin Pet Store application. Follow the steps below to connect to the MongoDB pod and explore its contents.

#### **1- Access the MongoDB Shell**
Run the following command to connect to the MongoDB shell inside the running MongoDB pod:
```bash
kubectl exec -it <mongodb-pod-name> -- mongo
```
Explanation: This command uses kubectl exec to open an interactive shell (-it) inside the MongoDB pod and starts the MongoDB shell program (mongo).

#### **2- List All Databases**
Once inside the MongoDB shell, run:
```bash
show dbs
```
Explanation: The show dbs command lists all databases available on the MongoDB server. You should see a list that includes the orderdb, which stores order-related data for the application.
#### **3- Switch to the Order Database**
```bash
use orderdb
```
Explanation: The use orderdb command selects the orderdb database, making it the active database for subsequent queries and commands.
#### **4- List Collections in the Database**
Display all collections in the orderdb database:
```bash
show collections
```
Explanation: The show collections command lists all collections (similar to tables in relational databases) in the current database. The orders collection contains the order data.
#### **5- Query the Orders Collection**
Retrieve all documents in the orders collection:
```bash
db.orders.find()
```
Explanation: The db.orders.find() command fetches and displays all documents (records) in the orders collection. This allows you to view the stored order data, including details such as customer information, products, and order status.

#### By following these steps, you will:
- Connect to the MongoDB shell in the Kubernetes pod.
- Explore the databases and collections used by the application.
- Query the orders collection to examine the data structure and stored records.

## Microservices Repository Table
This table lists the GitHub repositories for each of the microservices in the project:

| **Service**        | **Repository Link**                          |
|--------------------|----------------------------------------------|
| Store-Front        | [store-front-bestbuy](https://github.com/dhruv-13/store-front-L8)  |
| Order-Service      | [order-service-bestbuy](https://github.com/dhruv-13/order-service-L8) |
| Product-Service    | [product-service-bestbuy](https://github.com/dhruv-13/product-service-L8) |
| Makeline-Service   | [makeline-service-bestbuy](https://github.com/dhruv-13/makeline-service-L8) |
| Store-Admin        | [store-admin-bestbuy](https://github.com/dhruv-13/store-admin-L8) |
| AI-Service         | [AI-Service-bestbuy](https://github.com/dhruv-13/ai-service-L8) |
| virtual-worker     | [virtual-worker-bestbuy](https://github.com/dhruv-13/virtual-worker-L8) |
| Virtual-customer         | [virtual-customer-bestbuy](https://github.com/dhruv-13/virtual-customer-L8) |

# Docker Images Table

The following table provides links to the Docker Hub repositories for each of the microservices' Docker images:

| **Service**        | **Docker Image Link**                                          |
|--------------------|---------------------------------------------------------------|
| Store-Front        | [dhruv766/store-front](https://hub.docker.com/repository/docker/dhruv766/store-front/general) |
| Order-Service      | [dhruv766/order-service](https://hub.docker.com/repository/docker/dhruv766/order-service/general) |
| Product-Service    | [dhruv766/product-service](https://hub.docker.com/repository/docker/dhruv766/product-service/general) |
| Makeline-Service   | [dhruv766/makeline-service](https://hub.docker.com/repository/docker/dhruv766/makeline-service/general) |
| Store-Admin        | [dhruv766/store-admin](https://hub.docker.com/repository/docker/dhruv766/store-admin/general) |
| Ai-Service         | [dhruv766/ai-service](https://hub.docker.com/repository/docker/dhruv766/ai-service/general) |
| Virtual-worker         | [dhruv766/virtual-worker](https://hub.docker.com/repository/docker/dhruv766/virtual-worker/general) |
| Virtual Customer         | [dhruv766/virtual-customer](https://hub.docker.com/repository/docker/dhruv766/virtual-customer/general) |


MS Teams Link:

https://algonquinlivecom-my.sharepoint.com/:v:/g/personal/parm0100_algonquinlive_com/EWYLBfmWuE1NjIdw4ZvyFlMB0bdkJi2xVZzUVejo6dVmKg?e=2pv2gb&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D



