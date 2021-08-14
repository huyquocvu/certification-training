# Create and Configure Azure App Service
### Create an App Service
- [Create an App Service Plan](#1)
- [Configure scaling settings in an App Service plan](#2)
- [Create an App service](#3)
- [Secure an App service](#4)
### Configure App Services
- [Configure custom domain names](#5)
- [Configure backup for an App Service](#6)
- [Configure networking settings](#7)
- [Configure deployment settings](#8)

## Create an App Service Plan {#1}
- Supports containerization and Docker
- Web app and App Service Plan needs to be in the same region
- App cloning is supported for Standard, Premium, and Isolated app service plans
[App Service Plans](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)

##### Creating an App Service Plan
```PowerShell
# Create resource group
az group create --name webapp-rg --location centralus

# Create app service plan
az appservice plan create --name az104plan --resourcegroup webapp-rg --location centralus --sku S1 --is-linux
```

## Configure Scaling Settings in App Service Plan {#2}
- Scale up/down
- Scale in/out

##### Manually scaling an App Service Plan
```PowerShell
# Create a Web App in the App Service Plan
New-AzWebApp -Name testapp -ResourceGroupName webapp-rg -Location centralus -AppServicePlan az104plan

# Scale Web App to 2 Workers
Set-AzAppServicePlan -NumberofWorkers 2 -Name az104plan -ResourceGroupName webapp-rg
```
### Autoscaling
- Run the right number of resources to handle various loads
- Add resources to handle increased load
- Remove idle resources and save money
- Scale based on a schedule

## Create an App Service {#3}
- You **CANNOT** mix Windows and Linux apps in the same App Service plan
- Supports most languages
- .Net Core is supported on both Windows and Linux
- Optimized for DevOps

## Secure an App Service {#4}
- SSL certificate
- Authentication
    - EasyAuth
    - B2C
    - Social media accounts
    - SSO
    - MFA
- Access restriction
- Encryption using managed keys

### Certificate requirements
- App Service managed cert and App Service cert must meet requirements
- Must be exported as a password protected PFX file
- Contain private key at least 2048 bits long
- Contains all intermediate certificates in the cert chain

#### Authentication
- App Service provides built-in authentication and authorization
- Built-in ensures your solution stays up to date
- Built-in integrates with multiple login providers. Examples:
    - Azure AD
    - Facebook
    - Google
    - Twitter

#### Access Restrictions
- Define priority ordered allow/deny list
- Lists can include IPs or Azure Virtual Network subnets
- Works with all Azure App Service hosted workloads
- Service endpoints must be enabled on network and service side

|   |   |
| - | - |
| **<ul><li>Access restriction on Azure vNets is enabled by service endpoints</li><li>Service endpoints allow you to restrict access to a multi-tenant service</li><li>Does not work to restrict traffic to apps that are hosted in an App Service Environment</li></ul>** | ![Access Restriction](https://docs.microsoft.com/en-us/azure/app-service/media/app-service-ip-restrictions/access-restrictions-flow.png)|

### Encrypting using Managed Keys
- Encrypting a web app's data requires a storage account and Key Vault
- App Service can securely access secrets through a managed identity
- Revoke web app data access by rotating SAS key or removing apps access to Key Vault

## Configure Custom Domain Names {#5}
- Must be on a paid tier
- Must verify ownership of the domain by adding a verification ID
    - Completed by adding a TXT record to the domain provider
- For a root domain, add an (A) record at the domain provider

## Configure Backup for an App Service {#6}
- App configuration, file content, and database connected to app
- Can backup manually or scheduled
- Can perform partial and full backups
- Backups are visible on the containers page of storage account

### Requirements and Restrictions
- Must have Standard, Premium, or Isolated App Service Plan
- Must have an Azure storage account and container in same subscription
- Backups max out at 10 GB of app and database content
- Backups of TLS enabled Azure database for MySQL and PostgreSQL are not supported

## Configure Networking Settings {#7}
Network Options
- vNet Integration
- Hybrid Connections
- Azure Front Door with Web Application Firewall
- Azure CDN

### vNet Integration
- Multitenant and App Service Environment
    - Multitenants supports all pricing plans except Isolated
- Apps in App Service Environment do not require vNet integration
- vNet integration does not grant inbound private access to app from vNet
- Requires Standard, Premium, v2, v3, or Elastic Premium plan

### Hybrid Connections
- Both a service in Azure and a feature in Azure App Service
- Used to access application resources in *any* network that can make outbound calls to Azure over port 443
- Each Hybrid Connection correlates to a single TCP hot and port combination
- Need a Basic or higher App Service plan
    - Basic: 5 connections
    - Standard: 25 connections
    - Premium and Isolated: up to 220 connections

| | |
| - | - |
| **<ol><li>Hybrid connections require a relay agent</li><li>Relay agent and Hybrid Connect Manager makes a call over 443</li><li>On the app side, App Service also connects to the relay agent</li><li>Once joined, the app can access teh desired endpoint</li></ol>** | ![](https://docs.microsoft.com/en-us/azure/app-service/media/app-service-hybrid-connections/hybridconn-connectiondiagram.png) |

**Reference:https://bit.ly/3x35IF9**

### Azure Front Door with Web Application Firewall
- Azure Front Door works at Layer 7 using anycast protocol
- Routes client request to the fastest most available app backend
- Enables you to define, manage, and monitor ***global*** routing
- Can be configured using the Azure Portal, Azure CLI, PowerShell, Resource Manager Templates and REST APIs

| Azure Front Door directs web traffic to specific resources in backend pools that are in different regions | ![Azure Front Door](https://docs.microsoft.com/en-us/azure/frontdoor/media/front-door-overview/front-door-visual-diagram.png) |
| - | - |

### Azure CDN
- A content deliver network (CDN) is a distributed network of servers that can efficiently deliver web content to users
- Requires an Azure subscription and at least one CDN profile
- Can create multiple CDN profiesl based on different criteria
- CDN limitations include number of profiles, endpoints per profile and custom domains per endpoint
    - **25** CDN profiles
    - **25** endpoints per profile
    - **25** custom domains per endpoint, per subscription

| | |
| - | - |
| **<ol><li>Alice requests a file</li><li>DNS routes the request to the best performing POP location</li><li>Edge servers check cache. If not there, it requests from origin server</li><li>Origin server returns to edge server></li><li>Edge server returns file to Alice</li></ol>** | ![Azure CDN](https://docs.microsoft.com/en-us/azure/cdn/media/cdn-overview/cdn-overview.png) |

## Configure Deployment Settings {#8}
- Deployment source is the location of your code
- Build pipeline reads source code from the deployment source
- Staging environments include deployment slots which are based on your App Service Plan
- Continuous deployment should never be enabled for your production slot

