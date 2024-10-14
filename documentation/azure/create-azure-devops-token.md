# Steps to Azure DevOps Personal Access Token (PAT)
1. Sign in to Azure DevOps:

Go to the Azure DevOps portal: https://dev.azure.com.

Sign in with your Microsoft account credentials.

2. Navigate to the User Settings:

Once you're signed in, click on your profile picture in the top right corner of the Azure DevOps portal.
From the dropdown menu, select Security.

![](./assets/Screenshot%202024-10-08%20194736.png)

3. Generate a Personal Access Token (PAT):

In the Security section, under Personal access tokens, click on the + New Token button to generate a new token.

![](./assets/Screenshot%202024-10-08%20194957.png)

4. Configure the Token:

-   Name: Give the token a name to describe its purpose.
-   Organization: Select the organization where the token will be used.
-   Scopes: Select the appropriate scopes (permissions) for the token. Scopes determine what the token 
-   Expiration: Choose how long the token will be valid for (recommended to use the shortest period required for security reasons).


5. Create the Token:

Once you’ve configured the options, click the Create button.
Copy the Token:

After creating the token, Azure DevOps will display the token only once. Copy the token and store it in a secure location (e.g., password manager).

⚠️ Important: You won’t be able to view the token again after you leave this page, so make sure to save it securely.