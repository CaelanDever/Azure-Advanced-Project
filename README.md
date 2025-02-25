# Azure Advanced Project

---
Project Overview and Architecture

High-Level Diagram

Below is a high-level architecture you’ll build:


<img width="389" alt="adaz" src="https://github.com/user-attachments/assets/406ce59f-23eb-4f43-861c-bd94412accf6" />

                   
You will create multiple services in a well-structured and secure manner so that future code is set up and ready to be deployed in a CI/CD pipeline. This design is typical for an enterprise environment with both traditional IaaS components (like VMs) and modern PaaS components (like Azure Functions).

---
# Prerequisites and Setup

Azure Subscription: Ensure you have sufficient permissions (Owner or Contributor) or are using a free Azure account with a limit that can handle the project resources.

Azure CLI / PowerShell / Azure Portal: You can manage resources using any of these. We will showcase some key commands but feel free to use the Portal for a graphical approach.

Code Editor: VS Code (preferred) with Azure extensions or any editor you prefer.

---
# Step 1: Resource Organization (Management Group and Resource Group)
What You’ll Do

Create a Management Group (optional but recommended in enterprise environments).

Create a Resource Group named rg-azure-advanced within a region that meets your cost and compliance requirements.

---
Why This Matters

Management Groups let you apply policies and access controls at scale across multiple subscriptions.

Resource Groups logically group services for easier management, cost control, and life-cycle alignment.

---
Step 1 Instructions

Create a Management Group (optional if you want to skip this):


az account management-group create --name "SeniorCloudMG"

Create a Resource Group:


az group create \
    --name rg-azure-advanced \
    --location eastus

Best Practices

Choose a region close to your users or compliance boundaries (e.g., if data residency is a concern).

Include region in your resource naming if you follow a strict naming convention (e.g., rg-azure-advanced-eastus).

Screenshot:


<img width="824" alt="1w" src="https://github.com/user-attachments/assets/ce034bb2-cbb7-47ca-ab18-9e5249df7b8a" />

---
# Step 2: Network Design (VNet, Subnets, and NSGs)

What You’ll Do

Create a Virtual Network (VNet).

Carve out multiple subnets (at least two for separation of concerns).

Attach a Network Security Group (NSG) to control inbound and outbound traffic rules.

---
Why This Matters

Proper network segmentation is critical in enterprise environments to isolate workloads.

NSGs are your primary firewall at the subnet or NIC level, ensuring only allowed traffic flows through.

Step 2 Instructions

Create a Virtual Network:

az network vnet create \
    --resource-group rg-azure-advanced \
    --name vnet-advanced \
    --address-prefix 10.0.0.0/16 \
    --subnet-name snet-apps \
    --subnet-prefix 10.0.1.0/24

Add Another Subnet (e.g., for database or internal services):


az network vnet subnet create \
    --resource-group rg-azure-advanced \
    --vnet-name vnet-advanced \
    --name snet-db \
    --address-prefixes 10.0.2.0/24

Create a Network Security Group:


az network nsg create \
    --resource-group rg-azure-advanced \
    --name nsg-advanced
Associate NSG with Subnet:


az network vnet subnet update \
    --resource-group rg-azure-advanced \
    --vnet-name vnet-advanced \
    --name snet-apps \
    --network-security-group nsg-advanced

Create NSG Rules (allow SSH or RDP from specific IP ranges):

az network nsg rule create \
    --resource-group rg-azure-advanced \
    --nsg-name nsg-advanced \
    --name AllowSSH \
    --priority 1000 \
    --source-address-prefixes <YourIP>/32 \
    --destination-port-ranges 22 \
    --access Allow \
    --protocol Tcp \
    --description "Allow SSH from admin IP"

Best Practices

Use Azure’s Well-Architected Framework guidelines for network segmentation.

Keep your address spaces organized and sufficiently large but also be mindful not to overlap with on-premises networks if connecting via VPN/ExpressRoute in future expansions.

Screenshot:

<img width="653" alt="3we" src="https://github.com/user-attachments/assets/7ebf579b-28b5-44d0-bd82-3b001e632ca0" />

<img width="719" alt="4we" src="https://github.com/user-attachments/assets/bde1ad72-9f57-4f3e-a488-452092535b23" />

<img width="717" alt="5w" src="https://github.com/user-attachments/assets/5effacf9-74bb-47c7-a831-6c88a9b6fffd" />

---
# Step 3: Compute Layer (Azure VMs / Scale Sets)

What You’ll Do

Create an Azure VM (or a VM Scale Set for better scalability).

Install a simple web application or service on the VM.

---
Why This Matters

Demonstrates typical IaaS workload hosting (legacy or custom software).

VM Scale Sets are used for auto-scaling in enterprise environments to handle load variations.

Step 3 Instructions

Create an Azure VM:

az vm create \
    --resource-group rg-azure-advanced \
    --name vm-frontend \
    --image UbuntuLTS \
    --size Standard_B2s \
    --admin-username azureuser \
    --authentication-type ssh \
    --vnet-name vnet-advanced \
    --subnet snet-apps \
    --public-ip-address vm-frontend-ip

Connect to VM:


ssh azureuser@<PublicIPAddress>
Install Web Server (e.g., Nginx):

sudo apt-get update && sudo apt-get install -y nginx

sudo systemctl enable nginx

sudo systemctl start nginx

Best Practices

VM Scale Sets: For real-world large-scale scenarios, consider using Azure VM Scale Sets.

Automated Patching and Configuration: Use Azure Automation or cloud-init scripts for initialization and compliance.

Screenshot:

<img width="719" alt="6w" src="https://github.com/user-attachments/assets/2e24e830-ffe6-4ec2-b3fb-08a86dc4e062" />

---

# Step 4: Serverless Layer (Azure Function)

What You’ll Do

Create a serverless Azure Function (HTTP triggered).

Configure it to interact with Azure Storage, or a database, or external APIs.

---
Why This Matters

Serverless is cost-effective and scales dynamically.

Perfect for event-driven workloads or backend processing.

Step 4 Instructions

Create a Storage Account (Functions require a storage account):

az storage account create \
    --resource-group rg-azure-advanced \
    --name saazureadvanced \
    --location eastus \
    --sku Standard_LRS

Create a Function App:


az functionapp create \
    --resource-group rg-azure-advanced \
    --name fn-azure-advanced \
    --storage-account saazureadvanced \
    --consumption-plan-location eastus \
    --runtime dotnet

<img width="945" alt="11w" src="https://github.com/user-attachments/assets/c884d5de-e375-44d0-a84c-90a9fbf000d3" />


Adjust --runtime to node, python, or dotnet based on your preference.

Develop Your Function

In VS Code, install the Azure Functions Extension.

Create a new Project -> Select language -> Pick an HTTP Trigger template.

Deploy the Function

In VS Code, right-click on the Azure Functions project -> “Deploy to Function App” -> select your fn-azure-advanced.

Test the Function

Navigate to the function’s endpoint: https://fn-azure-advanced.azurewebsites.net/api/<FunctionName>?name=AzureTest

Or use curl / Postman to hit the endpoint.

Best Practices

Consumption Plan: Minimal cost for infrequent usage, scales automatically. For enterprise-level, consider Premium Plan if you need VNet integration, no cold starts, etc.

Store secrets (e.g., connection strings) in Azure Key Vault and reference them via Function App settings.

Screenshot:

<img width="765" alt="2wr" src="https://github.com/user-attachments/assets/e8587d97-5052-4272-b5ed-167b0a5885af" />

<img width="892" alt="httptrigger" src="https://github.com/user-attachments/assets/1ad33c84-9c78-4768-b5b8-2b963eb24128" />

<img width="892" alt="httptrigger" src="https://github.com/user-attachments/assets/36de493f-5770-4258-b3a1-2c9c6e46e28a" />

---

# Step 5: Storage and Database (Azure Storage + Azure SQL Database or Cosmos DB)

What You’ll Do

Connect your function or VM-based app to a database (either Azure SQL or Cosmos DB).

Optionally store files in Blob Storage or use Azure Files if needed.

---
Why This Matters

Data persistence and management are critical in virtually any enterprise application.

Using a managed database offloads patching, high availability, and backups to Azure.

Step 5 Instructions

Azure SQL Database:

az sql server create \
    --name sqlserveradvanced \
    --resource-group rg-azure-advanced \
    --location eastus \
    --admin-user sqladmin \
    --admin-password "<YourStrongPassword>"

az sql db create \
    --resource-group rg-azure-advanced \
    --server sqlserveradvanced \
    --name db-advanced \
    --service-objective S0

<img width="351" alt="azdb" src="https://github.com/user-attachments/assets/7ed03501-49ef-4b08-b36c-6ec49cb8e6c6" />

Adjust --service-objective to match performance needs.

Firewall Rule (Allow Azure services to connect):


az sql server firewall-rule create \
    --resource-group rg-azure-advanced \
    --server sqlserveradvanced \
    --name AllowAzureServices \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 0.0.0.0

Connection Strings

Retrieve via Portal or CLI, then set them as Function App settings or VM environment variables.

Best Practices

For global scale or non-relational scenarios, consider Cosmos DB.

Use private endpoints for secure, internal traffic to your DB from your VNet.

Screenshot:


<img width="562" alt="6we" src="https://github.com/user-attachments/assets/b6d65d88-ddeb-47a8-8314-6ec26ca83341" />

<img width="948" alt="dbadvanced" src="https://github.com/user-attachments/assets/8ff35e23-f74a-4ad5-9df0-e9c7b5592dac" />

---
# Step 6: Monitoring and Logging (Azure Monitor, App Insights, Log Analytics)

What You’ll Do

Configure Azure Monitor for metrics and alerts.

Enable Application Insights for your Azure Function (and possibly your VM apps).

Collect logs in Log Analytics workspace.

---
Why This Matters

Observability is crucial for diagnosing issues, tracking usage, and optimizing performance.

Step 6 Instructions

Create Log Analytics Workspace:

az monitor log-analytics workspace create \
    --resource-group rg-azure-advanced \
    --workspace-name la-advanced \
    --location eastus

Enable App Insights for your Function:

az monitor app-insights component create \
    --app fn-azure-advanced-ai \
    --resource-group rg-azure-advanced \
    --location eastus \
    --kind web

Link Function App to App Insights:

az resource update \
    --resource-group rg-azure-advanced \
    --name fn-azure-advanced \
    --resource-type "Microsoft.Web/sites" \
    --set properties.siteConfig.appSettings=[
      {"name":"APPINSIGHTS_INSTRUMENTATIONKEY","value":"<Instrumentation_Key>"},
      {"name":"APPLICATIONINSIGHTS_CONNECTION_STRING","value":"<Connection_String>"}
    ]

Alerts:

Configure an alert rule for high memory usage or failed function executions in the Azure Portal or via CLI.

Best Practices

Use Azure Monitor best practices to set up proactive monitoring, anomaly detection, and dashboards.

Consider using Grafana or Power BI for advanced visualization.

Screenshot:

<img width="954" alt="appinsights" src="https://github.com/user-attachments/assets/686dade2-8d75-4d4e-8b52-16b3866de3ae" />


<img width="707" alt="logs" src="https://github.com/user-attachments/assets/0a818e29-02d6-4269-9884-00f7f410f54e" />


---
# Step 7: Governance and Security (Azure Policy, Azure Key Vault, RBAC)

What You’ll Do

Apply Azure Policy to enforce governance (e.g., naming conventions, region restrictions).

Use Azure Key Vault to store secrets, connection strings, and certificates.

Configure Role-Based Access Control (RBAC) to ensure least-privilege access.

---
Why This Matters

Enterprise compliance and security posture require consistent guardrails.

Protecting secrets is mandatory to avoid data breaches.

Step 7 Instructions

Create Azure Key Vault:

az keyvault create \
    --name kv-advanced \
    --resource-group rg-azure-advanced \
    --location eastus

    <img width="947" alt="keyvault" src="https://github.com/user-attachments/assets/8c993f11-1454-462a-9d09-b07c534c23c6" />

Add a Secret:

az keyvault secret set \
    --vault-name kv-advanced \
    --name "DbConnectionString" \
    --value "<YourDBConnectionString>"

Use Key Vault Reference in Function App:

In Configuration -> New Setting -> Set the value to @Microsoft.KeyVault(SecretUri=https://kv-advanced.vault.azure.net/secrets/DbConnectionString/...).

Azure Policy (Portal or CLI):

Assign built-in policies (e.g., “Allowed locations”) to enforce resource creation only in eastus.

Audit or deny non-compliant resources.

RBAC:

Use Azure RBAC roles to ensure only authorized users can deploy or manage resources.

Best Practices

Store all sensitive information in Key Vault, not in code or environment variables.

Regularly rotate secrets and credentials.

Apply the Principle of Least Privilege across your subscription and resources.

Screenshot:


<img width="715" alt="keyvaulto" src="https://github.com/user-attachments/assets/7ee2a47c-c61f-482d-9a95-0c17400377ac" />

---

# Step 8: Cost Optimization (Budgets, Azure Advisor)

What You’ll Do

Configure an Azure Budget alert for your subscription.

Use Azure Advisor recommendations to optimize unused or underutilized resources.

---
Why This Matters

Cost is a major concern in enterprise cloud environments.

Proactive alerts help you stay within budget and right-size resources.

Step 8 Instructions

Create a Budget:

In the Azure Portal, go to Cost Management + Billing -> Budgets -> Create.

Set a monthly limit (e.g., $100) and an alert threshold (e.g., 80%).

Review Azure Advisor:

In the Portal, search for “Advisor”.

Check recommendations under Cost, Security, Reliability, etc.

Best Practices

Use Azure Reservations if you have predictable workloads.

Automate power down of VMs during off-hours if possible.

Screenshot:

<img width="942" alt="budget" src="https://github.com/user-attachments/assets/eee4b39d-ea9f-4575-ab98-3d1a05480c2f" />


<img width="938" alt="advisor" src="https://github.com/user-attachments/assets/9830a6ab-b0b1-4d1d-8c32-f3e63fef8978" />

---


# Troubleshooting Approaches

Connection Issues (VNet, NSG):

Verify NSG rules, route tables, and IP addresses.

Check logs in Azure Network Watcher.

Function App Not Triggering:

Ensure the trigger type is correct (HTTP, Timer, Event).

Check Function logs in App Insights or the Azure Portal.

Performance Bottlenecks:

Use Application Insights Profiler and Azure Monitor metrics.

Scale up or out with appropriate tiers (e.g., Premium Function Plan, VM Scale Sets).

Cost Overruns:

Check budgets, reduce resource size, or switch to consumption-based services.

Regularly assess via Azure Advisor.

---
# Conclusion

By following this advanced Azure cloud project:

You’ve built an enterprise-grade solution using Azure VNet, NSG, Azure VMs/Scale Sets, serverless Azure Functions, Azure Storage, Azure SQL Database, Key Vault, Azure Monitor, and more.

You’ve set up the code ready to be integrated a CI/CD pipeline (Azure DevOps or GitHub Actions) and it will have ensured governance, security, and cost optimization best practices.

This end-to-end project mirrors many responsibilities of a senior cloud engineer—from architecture design and security to deployment automation and cost governance. By mastering these concepts, you’ll demonstrate both breadth and depth of expertise in Azure.
