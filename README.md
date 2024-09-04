# Full-Stack FastAPI and React Template

Welcome to the Full-Stack FastAPI and React template repository. This repository serves as a demo application for interns, showcasing how to set up and run a full-stack application with a FastAPI backend and a ReactJS frontend using ChakraUI.

## Project Structure

The repository is organized into two main directories:

- **frontend**: Contains the ReactJS application.
- **backend**: Contains the FastAPI application and PostgreSQL database integration.

Each directory has its own README file with detailed instructions specific to that part of the application.

## Getting Started

To get started with this template, please follow the instructions in the respective directories:

- [Frontend README](./frontend/README.md)
- [Backend README](./backend/README.md)

This repository contains a full-stack application setup using Docker Compose. The application consists of a FastAPI backend, a Node.js frontend, a PostgreSQL database, an Adminer database management tool, an Nginx Proxy Manager, and an Nginx web server.

### This repository is configured to be able to deploy locally (development) and in production.

- **backend**: FastAPI application serving the backend API.
- **frontend**: Node.js application serving the frontend.
- **db**: PostgreSQL database for storing application data.
- **adminer**: Database management tool to interact with the PostgreSQL database.
- **nginx_proxy**: Nginx Proxy Manager to handle SSL certificates and domain management both locally and in production


## How to set up locally

1. **Clone the repository**:

   ```sh
   git clone https://github.com/DrInTech22/devops-stage-2.git
   cd devops-stage-2
   ```

2. **Build and start the services**:

   ```sh
   docker compose up -d
   ```
   for older version of docker compose, run:
   ```sh
   docker-compose up -d
   ```   

3. **Verify the services are running and all path are accessible**:
   - **FastAPI Backend**: [http://localhost/api](http://localhost/api)
   - **FastAPI Backend Docs**: [http://localhost/docs](http://localhost/docs)
   - **FastAPI Backend Redoc**: [http://localhost/api/redoc](http://localhost/api)
   - **Node.js Frontend**: [http://localhost](http://localhost)
   - **PostgreSQL Database**: internally ccessible on port `5432` (no direct browser access)
   - **Adminer**: [http://localhost:8080](http://localhost:8080) or [http://db.localhost](http://db.localhost)
   - **Nginx Proxy Manager**: [http://localhost:8090](http://localhost:8090) or [http://proxy.localhost](http://proxy.localhost)


## Local Service Details (docker-compose.yml)

### Backend (FastAPI)

- **Directory**: `./backend`
- **Dockerfile for development**: `Dockerfile.dev`
- **Port**: `8000`
- **Development Environment Variables file**: `backend/.env.dev`
  
### Frontend (Node.js)

- **Directory**: `./frontend`
- **Dockerfile for development**: `Dockerfile.dev`
- **Port**: `5173`
- **Development Environment Variables file**: `frontend/.env.dev`

### Database (PostgreSQL)

- **Docker Image**: `postgres:13`
- **Port**: `5432`
- **Environment Variables**:
  - `POSTGRES_USER`: Username for PostgreSQL.
  - `POSTGRES_PASSWORD`: Password for the PostgreSQL user.
  - `POSTGRES_DB`: Name of the PostgreSQL database.
- **Volumes**:
  - `postgres_data`: Persists PostgreSQL data.

### Adminer

- **Docker Image**: `adminer`
- **Port**: `8080`

### Proxy (Nginx Proxy Manager)

- **Docker Image**: `jc21/nginx-proxy-manager:latest`
- **Docker Container**: `nginx_proxy_manager`
- **Port**: `8090`
- **Volumes**:
  - `data`: Persistent data for the proxy manager.
  - `letsencrypt`: SSL certificates.
  - `./nginx/nginx.dev.conf`: Config for proxy manager.
- **Depends On**:
  - `frontend`
  - `backend`
  - `db`
  - `adminer`



## How to set up in production with domain
This section sets up the full stack application in production, configures domain name to access the application and secures it with ssl certificates.

### Set up domain
- Get a domain e.g mydomain.com, configure the following subdomain as A records pointing to your public ip.
- **mydomain.com, db.mydomain.com, proxy.mydomain.com**
- replace the subdomain in nginx/nginx.conf with your subdomain in all required lines.
- This example used **drintech.cloudopsdomain.online, db.drintech.cloudopsdomain.online, proxy.drintech.cloudopsdomain.online**

## Initial setup 
- clone the project
  ```sh
   git clone https://github.com/DrInTech22/devops-stage-2.git
   cd devops-stage-2
   ```
- run the project
  ```sh
  docker-compose -f docker-compose.prod.yml up -d
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
  - **drintech.cloudopsdomain.online**
  - **db.drintech.cloudopsdomain.online**
  - **proxy.drintech.cloudopsdomain.online**
- stop the applications
  ```
  docker-compose -f docker-compose.prod.yml down
  ```
## Final setup
- uncomment `#- ./nginx/nginx.prod.conf:/data/nginx/custom/http_top.conf` in the `docker-compose.prod.yaml` file. This maps nginx.prod.conf file on NPM.
```
volumes:
  - data:/data
  - letsencrypt:/etc/letsencrypt
#-./nginx/nginx.prod.conf:/data/nginx/custom/http_top.conf
```
- nginx.prod.conf sets up proxy host for the sub-domains, **www to non-www redirection** and **http to https redirection**.
- restart the application
  ```sh
  docker-compose -f docker-compose.prod.yml up -d
  ```
- **Verify the services are running and all path are accessible**:
   - **FastAPI Backend**: drintech.cloudopsdomain.online/api
   - **FastAPI Backend Docs**: drintech.cloudopsdomain.online/docs
   - **FastAPI Backend Redoc**: drintech.cloudopsdomain.online/redoc
   - **Node.js Frontend**: drintech.cloudopsdomain.online
   - **Adminer**: db.drintech.cloudopsdomain.online
   - **Nginx Proxy Manager**: proxy.drintech.cloudopsdomain.online

- test http to https redirection and www to non-www redirection are working
  - www.drintech.cloudopsdomain.online
  - https://www.drintech.cloudopsdomain.online
  - http://www.drintech.cloudopsdomain.online

## Production Service Details (docker-compose.prod.yml)

### Services

- **nginx-proxy**: Manages the proxying of traffic between different services.
  - Image: `jc21/nginx-proxy-manager:2.10.4`
  - Ports: `80:80`, `443:443`, `8090:81`
  - Environment:
    - `DB_SQLITE_FILE`: Path to the SQLite database file.
  - Volumes:
    - `data`: Persistent storage for Nginx Proxy Manager data.
    - `letsencrypt`: Persistent storage for Let's Encrypt certificates.
    - `./nginx/nginx.prod.conf`: config for proxy manager
  - Depends on: `frontend`, `backend`, `db`, `adminer`
  - Networks: `frontend-network`, `backend-network`, `db-network`

- **frontend**: The frontend service built from a custom Dockerfile.
  - Build context: `./frontend`
  - Dockerfile: `Dockerfile.prod`
  - Environment file: `frontend/.env.prod`
  - Depends on: `backend`
  - Networks: `frontend-network`, `backend-network`

- **backend**: The backend service built from a custom Dockerfile.
  - Build context: `./backend`
  - Dockerfile: `Dockerfile.prod`
  - Environment file: `backend/.env.prod`
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

- **adminer**: Database management tool.
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

- **frontend/.env.prod**: Contains production environment variables for the frontend service.
- **backend/.env.prod**: Contains production environment variables for the backend service.



