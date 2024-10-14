# Introduction
## Authentication in Backstage
The authentication system in Backstage serves two distinct purposes: sign-in and identification of users, as well as delegating access to third-party resources. It is possible to configure Backstage to have any number of authentication providers, but only one of these will typically be used for sign-in, with the rest being used to provide access to external resources.
## Built-in Authentication Providers
Backstage comes with many common authentication providers in the core library:

- Auth0
- Atlassian
- Azure
- Azure Easy Auth
- Bitbucket
- Bitbucket Server
- Cloudflare Access
- GitHub
- GitLab
- Google
- Google IAP
- Okta
- OAuth 2 Custom Proxy
- OneLogin
- VMware Cloud

These built-in providers handle the authentication flow for a particular service, including required scopes, callbacks, etc. These providers are each added to a Backstage app in a similar way.

# Custom Authentication in Backstage
![](../../assets/Screenshot%202024-10-08%20150546.png)

# Demo login using Azure Active Directory

![](../../assets/Screenshot%202024-10-08%20134131.png)

![](../../assets/Screenshot%202024-10-08%20150647.png)

![](../../assets/Screenshot%202024-10-08%20150732.png)

![](../../assets/Screenshot%202024-10-08%20150939.png)

![](../../assets/Screenshot%202024-10-08%20151024.png)

# Demo login using GitHub

![](../../assets/Screenshot%202024-10-08%20151307.png)

![](../../assets/Screenshot%202024-10-08%20151639.png)

![](../../assets/Screenshot%202024-10-08%20151731.png)

![](../../assets/Screenshot%202024-10-08%20151907.png)

![](../../assets/Screenshot%202024-10-08%20151847.png)

# Demo login using Google Cloud
![](../../assets/Screenshot%202024-10-08%20151307.png)

![](../../assets/Screenshot%202024-10-08%20164207.png)

![](../../assets/Screenshot%202024-10-08%20164327.png)