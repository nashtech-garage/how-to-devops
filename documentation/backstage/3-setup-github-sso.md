# Setup GitHub Authentication in your organization
1. Register an OAuth App in Github
![](./assets/Screenshot%202024-08-21%20072405.png)
![](./assets/Screenshot%202024-08-21%20072525.png)
After registering application you would have Client ID and Client secrets
2. Add Github provider into <i>app-config.yml</i>
Add below code under auth -> providers section
```yaml
    github:
      development:
        clientId: ${GITHUB_CLIENT_ID}
        clientSecret: ${GITHUB_CLIENT_SECRET}
        signIn:
          resolvers:
          # typically you would pick one of these
           # - resolver: usernameMatchingUserEntityName
            - resolver: emailMatchingUserEntityProfileEmail
          #- resolver: emailLocalPartMatchingUserEntityName
```
3. Using a <i>.env </i> File for Local Development
If you prefer not to set environment variables globally, you can create a .env file if is not existing in the root of your Backstage project directory and define the environment variables there. Example .env File
```makefile
    GITHUB_CLIENT_ID=your-github-client-id
    GITHUB_CLIENT_SECRET=your-github-client-secret   
```
4. Update <i>identityProviders.ts</i>.
 Create <i>identityProviders.ts</i> file folder in  packages/app/src/signin folder if it is not existing
```ts
    import { githubAuthApiRef } from '@backstage/core-plugin-api';
    export const providers = [
    {
        id: 'github-auth-provider',
        title: 'GitHub',
        message: 'Sign in using GitHub',
        apiRef: githubAuthApiRef
    }
    ];
```
5. Import <i>identityProvider</i> into App.tsx file if it has not imported yet
```ts
    import {providers} from './components/signin/identityProviders';
    components: {
    SignInPage: props => <SignInPage {...props} auto providers={providers} />
  },
```
6. Verify the application with yarn dev
![](./assets/Screenshot%202024-08-22%20003157.png)
![](./assets/Screenshot%202024-08-22%20003306.png)
The error message "Login failed; caused by Error: Failed to sign-in, unable to resolve user identity" suggests that the sign-in process is not able to properly resolve or match the user's identity after they have successfully authenticated with the identity provider.

You have to ensure that the user entity exists in the catalog or you can [Sign-In without Users in the Catalog](https://backstage.io/docs/auth/identity-resolver/#sign-in-without-users-in-the-catalog)

7. Optional- Sign-In without Users in the Catalog

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
       
        const providerConfigs = [
          {
            providerId: 'github',
            authenticator: githubAuthenticator
          },
          {
            providerId: 'microsoft',
            authenticator: microsoftAuthenticator
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
//backend.add(import('@backstage/plugin-auth-backend-module-microsoft-provider'));
// backend.add(import('@backstage/plugin-auth-backend-module-github-provider'));
backend.add(customAuth);
```

8. Verify the application with yarn dev
![](./assets/Screenshot%202024-08-22%20004823.png)
![](./assets/Screenshot%202024-08-22%20005048.png)