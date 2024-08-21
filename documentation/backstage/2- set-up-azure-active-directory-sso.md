# Setup Azure Active Directory SSO 
1. Register Application in Azure Active Directory
![](./assets/Screenshot%202024-08-20%20144407.png)
![](./assets/Screenshot%202024-08-20%20144708.png)
![](./assets/Screenshot%202024-08-20%20144856.png)
![](./assets/Screenshot%202024-08-20%20145711.png)
2. Add Microsoft provider into <i>app-config.yml</i>
Add below code under auth -> providers section
```yaml
    microsoft:
        development:
            clientId: ${AUTH_MICROSOFT_CLIENT_ID}
            clientSecret: ${AUTH_MICROSOFT_CLIENT_SECRET}
            tenantId: ${AUTH_MICROSOFT_TENANT_ID},
            signIn:
                resolvers:
                 # typically you would pick one of these
                    - resolver: emailMatchingUserEntityProfileEmail
                    #- resolver: emailLocalPartMatchingUserEntityName
                    #- resolver: emailMatchingUserEntityAnnotation 

```

3. Using a <i>.env </i> File for Local Development
If you prefer not to set environment variables globally, you can create a .env file in the root of your Backstage project directory and define the environment variables there. Example .env File
```makefile
    AZURE_CLIENT_ID=your-azure-client-id
    AZURE_CLIENT_SECRET=your-azure-client-secret
    AZURE_TENANT_ID=your-azure-tenant-id
```
4. Create <i>signin</i> component. First create signin folder in  packages/app/src/ folder, second create <i>identityProviders.ts</i> inside signin folder.
```ts
    import { microsoftAuthApiRef } from '@backstage/core-plugin-api';
    export const providers = [
    {
        id: 'azure-auth-provider',
        title: 'Azure Active Directory',
        message: 'Sign in using Azure AD',
        apiRef: microsoftAuthApiRef,
    },
    ];
```
![](./assets/Screenshot%202024-08-20%20160745.png)
5. Import <i>identityProvider</i> into App.tsx file
```ts
    import {providers} from './components/signin/identityProviders';
    components: {
    SignInPage: props => <SignInPage {...props} auto providers={providers} />,
  },
```
![](./assets/Screenshot%202024-08-20%20161111.png)

6. Verify the application with yarn dev
![](./assets/Screenshot%202024-08-20%20200337.png)
![](./assets/Screenshot%202024-08-20%20200528.png)
![](./assets/Screenshot%202024-08-21%20235353.png)

The error message "Login failed; caused by Error: Failed to sign-in, unable to resolve user identity" suggests that the sign-in process is not able to properly resolve or match the user's identity after they have successfully authenticated with the identity provider.

You have to ensure that the user entity exists in the catalog or you can [Sign-In without Users in the Catalog](https://backstage.io/docs/auth/identity-resolver/#sign-in-without-users-in-the-catalog)

7. Optional- Sign-In without Users in the Catalog
Create custom signin resolve
Update <i>indext.ts</i> file in  <i>/packages/backend/src/</i> folder
```ts
import { createBackendModule } from '@backstage/backend-plugin-api';
import { microsoftAuthenticator, } from '@backstage/plugin-auth-backend-module-microsoft-provider';
import { githubAuthenticator, } from '@backstage/plugin-auth-backend-module-github-provider';
import {
  authProvidersExtensionPoint,
  createOAuthProviderFactory,
} from '@backstage/plugin-auth-node';

import { stringifyEntityRef, DEFAULT_NAMESPACE } from '@backstage/catalog-model';
const customAuth = createBackendModule({
  // This ID must be exactly "auth" because that's the plugin it targets
  pluginId: 'auth',
  // This ID must be unique, but can be anything
  moduleId: 'custom-auth-provider',
  register(reg) {
    reg.registerInit({
      deps: { providers: authProvidersExtensionPoint },
      async init({ providers }) {
        providers.registerProvider({
          providerId: 'microsoft',      
          factory: createOAuthProviderFactory({
            authenticator: microsoftAuthenticator,
            async signInResolver({ profile }, ctx) {
              if (!profile.email) {
                throw new Error(
                  'Login failed, user profile does not contain an email',
                );
              }
              // Split the email into the local part and the domain.
              const [localPart] = profile.email.split('@');
        
              // By using `stringifyEntityRef` we ensure that the reference is formatted correctly
              const userEntity = stringifyEntityRef({
                kind: 'User',
                name: localPart,
                namespace: DEFAULT_NAMESPACE,
              });
              return ctx.issueToken({
                claims: {
                  sub: userEntity,
                  ent: [userEntity],
                },
              });
            },
            
          }),
        });
      },
    });
  },
});
//backend.add(import('@backstage/plugin-auth-backend-module-microsoft-provider'));
backend.add(customAuth);
```
8. Verify the application with yarn dev

![](./assets/Screenshot%202024-08-20%20200337.png)
![](./assets/Screenshot%202024-08-20%20200528.png)
![](./assets/Screenshot%202024-08-22%20002146.png)
![](./assets/Screenshot%202024-08-22%20002038.png)