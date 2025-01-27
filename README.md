# Deploying ResumeAI (MEAN Application) on Azure with Monitoring Dashboard

This README provides a comprehensive guide to deploying the ResumeAI project using Azure services. It also includes steps to set up a monitoring dashboard to track application metrics.

---

## Prerequisites

Before starting, ensure you have the following:

1. An Azure account ([create a free account here](https://azure.microsoft.com/free/)).
2. Azure CLI installed ([installation guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)).
3. Node.js and npm installed ([download here](https://nodejs.org/)).
4. Angular CLI installed globally:
   ```bash
   npm install -g @angular/cli
   ```
5. A GitHub repository to host your code (optional but recommended).

---

## Step 1: Set Up Azure Resources

### 1.1 Create a Resource Group

```bash
az group create --name ResumeAIResourceGroup --location eastus
```

### 1.2 Set Up an Azure Cosmos DB for MongoDB

```bash
az cosmosdb create --name ResumeAICosmosDB --resource-group ResumeAIResourceGroup --kind MongoDB --server-version 4.0
```

Retrieve the connection string for the database:

```bash
az cosmosdb keys list --type connection-strings --name ResumeAICosmosDB --resource-group ResumeAIResourceGroup
```

### 1.3 Create an Azure App Service Plan

```bash
az appservice plan create --name ResumeAIServicePlan --resource-group ResumeAIResourceGroup --sku P1V2 --is-linux
```

### 1.4 Deploy a Web App

Create separate web apps for the frontend and backend:

#### Frontend:
```bash
az webapp create --resource-group ResumeAIResourceGroup --plan ResumeAIServicePlan --name ResumeAIFrontend --runtime "NODE|16-lts"
```

#### Backend:
```bash
az webapp create --resource-group ResumeAIResourceGroup --plan ResumeAIServicePlan --name ResumeAIBackend --runtime "NODE|16-lts"
```

---

## Step 2: Deploy the ResumeAI Application

### 2.1 Prepare the Angular Frontend

1. Navigate to the `ResumeBuilderAngular` folder:
   ```bash
   cd ResumeBuilderAngular
   ```
2. Build the Angular application for production:
   ```bash
   ng build --prod
   ```
3. Zip the build output:
   ```bash
   zip -r frontend.zip dist/
   ```

### 2.2 Deploy the Frontend to Azure

1. Use Azure CLI to deploy the frontend:
   ```bash
   az webapp deployment source config-zip --resource-group ResumeAIResourceGroup --name ResumeAIFrontend --src frontend.zip
   ```

### 2.3 Prepare the Node.js Backend

1. Navigate to the `ResumeBuilderBackend` folder:
   ```bash
   cd ResumeBuilderBackend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Build the application:
   ```bash
   npm run build
   ```
4. Zip the build output:
   ```bash
   zip -r backend.zip dist/
   ```

### 2.4 Deploy the Backend to Azure

1. Use Azure CLI to deploy the backend:
   ```bash
   az webapp deployment source config-zip --resource-group ResumeAIResourceGroup --name ResumeAIBackend --src backend.zip
   ```

---

## Step 3: Configure Monitoring and Alerts

### 3.1 Enable Application Insights

#### Frontend:
```bash
az monitor app-insights component create --app ResumeAIFrontendInsights --location eastus --resource-group ResumeAIResourceGroup
az webapp config appsettings set --resource-group ResumeAIResourceGroup --name ResumeAIFrontend --settings "APPINSIGHTS_INSTRUMENTATIONKEY=<YOUR_FRONTEND_INSTRUMENTATION_KEY>"
```

#### Backend:
```bash
az monitor app-insights component create --app ResumeAIBackendInsights --location eastus --resource-group ResumeAIResourceGroup
az webapp config appsettings set --resource-group ResumeAIResourceGroup --name ResumeAIBackend --settings "APPINSIGHTS_INSTRUMENTATIONKEY=<YOUR_BACKEND_INSTRUMENTATION_KEY>"
```

### 3.2 Set Up Metrics and Alerts

1. Go to the [Azure Portal](https://portal.azure.com/).
2. Navigate to the "Application Insights" resource for both the frontend and backend.
3. Configure the following metrics:
   - Server response time
   - Request count
   - Failed requests
   - Dependency failures
4. Create alert rules for critical thresholds (e.g., server response time > 1 second).

---

## Step 4: Create a Monitoring Dashboard

### 4.1 Use Azure Monitor

1. Go to Azure Monitor in the Azure Portal.
2. Create a new dashboard.
3. Add the following widgets:
   - Application Insights: Live Metrics Stream (Frontend and Backend)
   - Metrics: Server Response Time, Request Count, Dependency Calls
   - Logs: Query logs for custom metrics.

### 4.2 Save and Share Dashboard

1. Save the dashboard under a recognizable name, e.g., `ResumeAI Monitoring`.
2. Share it with team members or stakeholders as needed.

---

## Step 5: Verify the Deployment

1. Access the frontend URL:
   ```
   https://<ResumeAIFrontend>.azurewebsites.net
   ```
2. Access the backend URL:
   ```
   https://<ResumeAIBackend>.azurewebsites.net
   ```
3. Test the application functionality.
4. Check the Application Insights logs for any errors.

---

## Additional Resources

- [Azure CLI Documentation](https://learn.microsoft.com/en-us/cli/azure/)
- [Azure App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/)
- [Application Insights Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

---

## Troubleshooting

1. **Deployment Fails**: Check the deployment logs for errors.
2. **Database Connection Issues**: Verify the Cosmos DB connection string and ensure the firewall rules allow access.
3. **Monitoring Not Working**: Ensure the Application Insights instrumentation key is correctly configured.

---
