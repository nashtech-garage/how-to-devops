# Run Backstage on Kubernetes

## Build the container image

1. In your application directory, build your application
```bash
    yarn install --frozen-lockfile
    yarn tsc
    yarn build:backend
```
2. Build the image
```bash
    docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
```
## Upload the image to your container registry 
You can upload the image to any your registry, in this case, Azure Container Register (ACR)
1. Login to ACR server
```bash
    docker login ntbackstage.azurecr.io
```
2. Tag backstage:1.0.0
```bash
    docker tag backstage:1.0.0 ntbackstage.azurecr.io/backstage:1.0.0
```
3. Push the image to <i>ntbackstage</i> container registry
```bash
    docker push ntbackstage.azurecr.io/backstage:1.0.0
```
