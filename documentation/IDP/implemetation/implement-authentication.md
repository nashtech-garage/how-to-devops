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

# Configuring Authentication Providers For Our IDP
Configuring Authentication for Azure Active Directory, GitHub and Google Cloud
![](../assets/Screenshot%202024-10-08%20134131.png)

## Add providers in app-config.yaml

```yaml
auth:
  # see https://backstage.io/docs/auth/ to learn about auth providers
  providers:
    microsoft:
      development:
        clientId: ${AZURE_CLIENT_ID}
        clientSecret: ${AZURE_CLIENT_SECRET}
        tenantId: ${AZURE_TENANT_ID}
       # signIn:
       #   resolvers:
          # typically you would pick one of these
           # - resolver: usernameMatchingUserEntityName
           # - resolver: emailMatchingUserEntityProfileEmail
          #- resolver: emailLocalPartMatchingUserEntityName
    github:
      development:
        clientId: ${GITHUB_CLIENT_ID}
        clientSecret: ${GITHUB_CLIENT_SECRET}
       # signIn:
         # resolvers:
          # typically you would pick one of these
           # - resolver: usernameMatchingUserEntityName
         #   - resolver: emailMatchingUserEntityProfileEmail
          #- resolver: emailLocalPartMatchingUserEntityName
    google:
      development:
        clientId: ${GOOGLE_CLIENT_ID}
        clientSecret: ${GOOGLE_CLIENT_SECRET}
```
## Create identityProvider.ts
Create identityProvider.ts in app/src/components/sigin

```ts
import { microsoftAuthApiRef , githubAuthApiRef , googleAuthApiRef } from '@backstage/core-plugin-api';

export const providers = [
  {
    id: 'microsoft-auth-provider',
    title: 'Azure Active Directory',
    message: 'Sign in using Azure AD',
    apiRef: microsoftAuthApiRef
  },
  {
    id: 'github-auth-provider',
    title: 'GitHub',
    message: 'Sign in using GitHub',
    apiRef: githubAuthApiRef
  },
  {
    id: 'google-auth-provider',
    title: 'Google',
    message: 'Sign in using Google',
    apiRef: googleAuthApiRef
  }
];
```

## Create customAuth in index.ts
In packages/backend/src/index.ts

```ts
const customAuth = createBackendModule({
  // This ID must be exactly "auth" because that's the plugin it targets

  pluginId: 'auth',
  // This ID must be unique, but can be anything
  moduleId: 'custom-auth-provider',
  register(reg) {
    reg.registerInit({
      deps: { providers: authProvidersExtensionPoint },
      async init({ providers }) {
       
        const providerConfigs = [
          {
            providerId: 'github',
            authenticator: githubAuthenticator
          },
          {
            providerId: 'microsoft',
            authenticator: microsoftAuthenticator
          },
          {
            providerId: 'google',
            authenticator: googleAuthenticator
          }
        ];

        providerConfigs.forEach(({ providerId, authenticator }) => {
          providers.registerProvider({
            providerId,
            factory: createOAuthProviderFactory({
              authenticator,
              async signInResolver({ profile }, ctx) {
                console.log(`start signInResolver`);
                if (!profile.email) {
                  throw new Error(
                    'Login failed, user profile does not contain an email',
                  );
                }
                // Split the email into the local part and the domain.
                const [localPart] = profile.email.split('@');          
                // By using `stringifyEntityRef` we ensure that the reference is formatted correctly
                const userEntityRef = stringifyEntityRef({
                  kind: 'User',
                  name: localPart.toLowerCase(),
                  namespace: DEFAULT_NAMESPACE,
                });                                               
                                        
                return ctx.issueToken({
                  claims: {
                    sub: userEntityRef,
                    ent: [userEntityRef]
                  },
                });


              },                  
              
            }),
          });
        });
          
      },
    });
  },
});

 backend.add(customAuth);
```
##  Defined variables in .env
```
export AZURE_CLIENT_ID=<azure-client-id>
export AZURE_CLIENT_SECRET=<azure-client-secret>
export AZURE_TENANT_ID=<azure-tenant-id>
export GITHUB_CLIENT_ID=<github-client-id>
export GITHUB_CLIENT_SECRET=<github-client-secret>
export GOOGLE_CLIENT_ID=<google-client-id>
export GOOGLE_CLIENT_SECRET=<google-client-secret>
```

# Generate Azure Client ID and Secret
Register Application in Azure Active Directory

![](../assets/Screenshot%202024-10-08%20144022.png)

![](../assets/Screenshot%202024-10-08%20144212.png)

![](../assets/Screenshot%202024-10-08%20144316.png)

  http://localhost:7007/api/auth/microsoft/handler/frame

![](../assets/Screenshot%202024-10-08%20144926.png)

![](../assets/Screenshot%202024-10-08%20144619.png)

# Generate GitHub Client ID and Secret
![](../assets/Screenshot%202024-10-08%20145351.png)

![](../assets/Screenshot%202024-10-08%20145446.png)

![](../assets/Screenshot%202024-10-08%20145608.png)

![](../assets/Screenshot%202024-10-08%20145756.png)

# Generate Google Client ID and Secret
Select Google Project and create Creadentials

![](../assets/Screenshot%202024-10-08%20150039.png)

![](../assets/Screenshot%202024-10-08%20150119.png)

![](../assets/Screenshot%202024-10-08%20150240.png)