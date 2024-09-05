# Full-Stack Application Deployment using Github Actions and Docker-compose

Welcome to the Full-Stack application deployment repository. This repository demonstrates how to automate and streamline the deployment of full stack react and fastapi applications using powerful tools such as github actions and docker compose. 

## Project Structure

The repository is organized into two main directories:

- **frontend**: Contains the ReactJS application.
- **backend**: Contains the FastAPI application and PostgreSQL database integration.

## Overview

This repository contains a full-stack application automated  deployment using Docker Compose and github actions. The application consists of a FastAPI backend, a Node.js frontend, a PostgreSQL database, an Adminer database management tool, an Nginx Proxy Manager.

Each of this application is containerized to facilitate easy deployment and scaling in respect to demand. The Github Actions CI/CD runs a CI pipeline that runs test on the fastAPI application. The CDs pipeline then continuously deploy code changes to frontend and backend containers on the staging environment. The deployments are accessible via custom domains as presented below.

<img src="assets/Screenshot (408).png" width="45%"></img> <img src="assets/Screenshot (407).png" width="45%"></img> 
<img src="assets/Screenshot (410).png" width="45%"></img> <img src="assets/Screenshot (405).png" width="45%"></img> 


## 
**I initially configured this project to be able to deploy locally (development) and in production before I integrated the github actions. You can check the full setup [here](https://github.com/DrInTech22/devops-stage-2)** 

- **backend**: FastAPI application serving the backend API.
- **frontend**: Node.js application serving the frontend.
- **db**: PostgreSQL database for storing application data.
- **adminer**: Database management tool to interact with the PostgreSQL database.
- **nginx_proxy**: Nginx Proxy Manager to handle SSL certificates and domain management both locally and in staging/production with an easily accessible interface.


## How to set up staging/production deployment with custom domain
This section sets up the full stack application in production, configures domain name to access the application and secures it with ssl certificates.

### Set up domain
- Get a domain e.g mydomain.com, configure the following subdomain as A records pointing to your public ip.
- **mydomain.com, db.mydomain.com, proxy.mydomain.com**
- replace the subdomain in nginx/nginx.staging.conf with your subdomain in all required lines.
- This example used **drintech.mooo.com, db.drintech.mooo.com, proxy.drintech.mooo.com**. 

## Initial setup 
- clone the project
  ```sh
   git clone https://github.com/DrInTech22/3-tier-prod-cicd.git
   cd 3-tier-prod-cicd
   ```
- run the project
  ```sh
  docker compose up -d
  ```
- access the NPM on your browser using your public-ip
  ```sh
   your_public_ip:8090
   ```
- login using NPM default login 
  ```
  username: admin@example.com
  password: changeme
  ```
  you will be prompted to change the password after login

- generate ssl certificates for your subdomains in the following **order below** using **lets encrypt**. We will use the sample domain.
  - **drintech.mooo.com**
  - **db.drintech.mooo.com**
  - **proxy.drintech.mooo.com**
## Final setup
- uncomment `#- ./nginx/nginx.staging.conf:/data/nginx/custom/http_top.conf` in the `compose.yaml` file. This maps nginx.prod.conf file on NPM.
```
volumes:
  - data:/data
  - letsencrypt:/etc/letsencrypt
#-./nginx/nginx.staging.conf:/data/nginx/custom/http_top.conf
```
- nginx.prod.conf sets up proxy host for the sub-domains, and configures **www to non-www redirection** and **http to https redirection**.
- recreate the containers
  ```sh
  docker compose up -d --force-recreate
  ```
- **Verify the services are running and all path are accessible**:
   - **FastAPI Backend Docs**: drintech.mooo.com/docs
   - **FastAPI Backend Redoc**: drintech.mooo.com/redoc
   - **Node.js Frontend**: drintech.mooo.com
   - **Adminer**: db.drintech.mooo.com
   - **Nginx Proxy Manager**: proxy.drintech.mooo.com

- test http to https redirection and www to non-www redirection are working
  - www.drintech.mooo.com
  - https://www.drintech.mooo.com
  - http://www.drintech.mooo.com

## Staging/Production Service Details (compose.yml)

### Services

- **nginx**: Manages the proxying of traffic between different services. It serve as the entrypoint of traffic into the containers. 
  - Image: `jc21/nginx-proxy-manager:2.10.4`
  - Ports: `80:80`, `443:443`, `8090:81`
  - Environment:
    - `DB_SQLITE_FILE`: Path to the SQLite database file.
  - Volumes:
    - `data`: Persistent storage for Nginx Proxy Manager data.
    - `letsencrypt`: Persistent storage for Let's Encrypt certificates.
    - `./nginx/nginx.prod.conf`: config for proxy manager
  - Depends on: `frontend`, `backend`, `db`, `adminer`
  - Networks: `frontend-network`, `backend-network`, 

- **frontend**: The frontend service built from a custom Dockerfile.
  - Build context: `./frontend`
  - Dockerfile: `Dockerfile`
  - Environment file: `frontend/.env.staging`
  - Depends on: `backend`
  - Networks: `frontend-network`, `backend-network`

- **backend**: The backend service built from a custom Dockerfile.
  - Build context: `./backend`
  - Dockerfile: `Dockerfile`
  - Environment file: `backend/.env.staging`
  - Environment:
    - `DATABASE_URL`: Connection string for the PostgreSQL database.
  - Depends on: `db`
  - Networks: `backend-network`, `db-network`

- **db**: PostgreSQL database service.
  - Image: `postgres:13`
  - Volumes:
    - `postgres_data`: Persistent storage for PostgreSQL data.
  - Environment:
    - `POSTGRES_DB`: Database name.
    - `POSTGRES_USER`: Database user.
    - `POSTGRES_PASSWORD`: Database password.
  - Ports: `5432`
  - Networks: `db-network` 
  - Only the backend and adminer can access the database, this adds another layer of security to the database.

- **adminer**: Database management tool. It provides an interface to access postgresql.
  - Image: `adminer`
  - Ports: `8080:8080`
  - Environment:
    - `ADMINER_DEFAULT_SERVER`: Default database server.
  - Networks: `db-network`

### Networks

- `frontend-network`: Network for frontend communication.
- `backend-network`: Network for backend communication.
- `db-network`: Network for database communication.

### Volumes

- `postgres_data`: Persistent storage for PostgreSQL.
- `data`: Persistent storage for Nginx Proxy Manager.
- `letsencrypt`: Persistent storage for Let's Encrypt certificates.

### Environment Files

- **frontend/.env.staging**: Contains staging environment variables for the frontend service.
- **backend/.env.staging**: Contains staging environment variables for the backend service.

## How to set Github Actions for Continuous Integration and Deployment
1. **backend-ci.yml - Continuous Integration (CI) Pipeline**:
This workflow handles the continuous integration for your application, triggered on pull requests and manual dispatch(optional). It sets up a test environment, installs dependencies, and runs tests.

   **Steps:**
- Set up postgresql services as a mock database.
- Set up Python and caches Python dependencies to speed up subsequent runs using actions/cache.
- Installs the required dependencies using Poetry and set the .env file
- Runs the app in the background and runs the test suite using pytest to verify the code functionality.

2. **frontend-ci.yml - Continuous Integration (CI) Pipeline**:
This workflow handles the continuous integration for your application, triggered on pull requests and manual dispatch(optional). It sets up a test environment, installs dependencies, lint and build the code.

    **Note: test script was not defined in package.json**

   **Steps:**
- Sets up a nodejs environment for testing the application
- Caches dependencies to speed up subsequent runs using actions/cache@v3
- Install dependencies, lint the code and build to verify the working functionality of the application.

3. **frontend-cd.yml - Frontend Continuous Deployment (CD)**
This workflow automates the deployment of the frontend application. It is triggered on push to the main branch when changes are made to the frontend folder.
- create an environment `frontend` and configure the following secrets and variables:
  - secrets
    - HOST
    - USERNAME
    - PASSWORD
  - variables
    - URL (to display deployment URL on workflow board)
  
```sh
docker compose up -d --no-deps --build frontend

```
The command above rebuilds the frontend container and makes sure all its depending containers are not restarted in the process. 

4. **backend-cd.yml - Backend Continuous Deployment (CD)**
This workflow automates the deployment of the backend application. It is triggered on push to the main branch when changes are made to the frontend folder.
- create an environment `backend` and configure the following secrets and variables:
  - secrets
    - HOST
    - USERNAME
    - PASSWORD
  - variables
    - URL (to display deployment URL on workflow board)
```sh
docker compose up -d --no-deps --build backend
```
The command above rebuilds the backend container and makes sure all its depending containers are not restarted in the process. 

<img src="assets/Screenshot (415).png" width="45%"></img> <img src="assets/Screenshot (414).png" width="45%"></img> 
