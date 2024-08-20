# Setup Backstage Development Environment

## Set up the repo

1. Fork backstage into your GitHub user [Backstage repo](https://github.com/backstage/backstage)

2. Clone the repo to your local workstation
```bash
    git clone git@github.com:backstage/backstage.git
```
3. cd to the backstage repo
4. Optional - checkout to latest statble version
```bash
    git checkout v1.29.2
```
## Install  Configure NVM

NVM - Node Version Manager, nvm allows you to quickly install and use different versions of Node via the command line.
1. Install NVM
```bash
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

2. Install Node 18
```bash
    nvm install 18
```
3. Use Node version 18
```bash
    nvm use 18
```
4. Set 18 as the default Node version
```bash
    nvm alias default 18
```
## Install Yarn
1. Install Yarn
```bash
    npm install --global yarn
```
2. Set Yarn to version 1
```bash
    yarn set version 1.22.19
```
3. Verify the Yarn version
```bash
   yarn --version
```
## Create your Backstage Application

1. Create a Backstage application
```bash
   npx @backstage/create-app@latest
```
2. cd to your application directory ( based on the given name)
```bash
   cd nashtech-accelerator
```
3. Run backstage in a development mode
```bash
   yarn dev
```
## Verify your application

1. Go to the Backstage UI (should open automactically after the <i> yarn dev </i> command)
![](./assets/Screenshot%202024-08-19%20161150.png)
![](./assets/Screenshot%202024-08-19%20161258.png)
2. Import a component using the file by going to [Register an existing component](http://localhost:3000/catalog-import) and import this file- https://github.com/backstage/backstage/blob/master/catalog-info.yaml
![](./assets/Screenshot%202024-08-19%20162139.png)
3. Review the example component
![](./assets/Screenshot%202024-08-19%20162452.png)
![](./assets/Screenshot%202024-08-19%20162431.png)
