# Steps to Create a Personal Access Token (PAT) on GitHub
To generate an Azure Active Directory (Azure AD) Token for authentication, such as for use in API requests or other Azure services, you typically use the Azure CLI or the Azure Portal.
1. Using Azure CLI:
The easiest and most common way to get an access token for Azure is by using the Azure CLI. Follow these steps:

Step-by-Step Instructions:
Install Azure CLI (if you haven't already):

You can install the Azure CLI by following Microsoft's official guide.
Log in to Azure: Once Azure CLI is installed, open your terminal and run:
```bash
az login

```
This will open a web browser window where you need to log in with your Azure credentials.
After logging in, your terminal will display information about your account, such as your subscription details.

Generate the Access Token: You can generate an access token for Azure Resource Manager (ARM) using the following command:
```bash
az account get-access-token

```
This will return a JSON response, and you can find the access token within it:
```json
{
  "accessToken": "eyJ0eXAiOiJKV1QiLCJh...",
  "expiresOn": "2024-07-01 13:45:12.000000",
  "subscription": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tokenType": "Bearer"
}

```