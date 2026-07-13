# Dream Vacation Destinations

This application allows users to create a list of countries they'd like to visit, providing basic information about each country. The project is structured to mimic a real-life production environment, employing best practices in software development, deployment, and continuous integration/continuous delivery (CI/CD).

## Setup

### Backend
1. Navigate to the `backend` directory.
2. Run `npm install` to install dependencies.
3. Set up your PostgreSQL database and update the `.env` file with your database URL.
4. Run `npm start` to start the server.

### Frontend
1. Navigate to the `frontend` directory.
2. Run `npm install` to install dependencies.
3. Update the `.env` file with your API URL (e.g., `REACT_APP_API_URL=http://localhost:3001`).
4. Run `npm start` to start the React development server.

## Features
- **Add Countries**: Users can add countries to their dream vacation list.
- **View Country Details**: Displays capital, population, and region information for each country.
- **Remove Countries**: Users can remove countries from their list.
- **Production-Ready Setup**: The project is designed to be scalable and maintainable, following industry-standard practices for deployment and CI/CD.

## Roadmap
- **CI/CD Implementation**: Automate the build, test, and deployment process using industry-standard CI/CD tools.
- **Infrastructure as Code (IaC)**: Implement IaC for automated environment setup and management.
- **Scalability**: Enhance the application to support multiple environments (staging, production) with proper domain names and configurations.
- **Security**: Utilize Kubernetes Secrets and environment variables for secure data management.
- **Microservices**: Modularize the application into microservices to improve maintainability and scalability.

## Technologies Used
- **Frontend**: React
- **Backend**: Node.js with Express
- **Database**: PostgreSQL
- **External API**: REST Countries API
- **CI/CD**: To be implemented with [CI/CD tools, e.g., GitHub Actions, Jenkins, or Azure DevOps]
- **Infrastructure as Code**: To be implemented with tools like Terraform or Helm

## Best Practices
- **Version Control**: All changes are tracked in Git for collaboration and history management.
- **Environment Management**: Separate configurations for different environments (development, staging, production) using environment variables.
- **Security**: Sensitive information is managed using environment variables and Kubernetes Secrets.
- **Documentation**: The project is well-documented to facilitate onboarding and maintenance.

## 🚀 CI/CD Pipeline Automation

This project utilizes **GitHub Actions** to automate the integration and delivery pipelines. The pipeline is split into micro-workflows (`frontend.yml` and `backend.yml`) that trigger independently based on where changes occur, minimizing unnecessary build minutes.

### Pipeline Architecture & Triggers
* **Branches Monitored:** `main`, `dev`
* **Trigger Events:** Any `push` or `pull_request` targeting the monitored branches.
* **Path Filtering:** The frontend pipeline only fires if files within the `/frontend` directory change; the backend pipeline behaves identically for the `/backend` directory.

### Multi-Stage Workflow Execution
1. **Test Stage (`test` job):** * Installs dependencies using optimized caching (`npm ci`).
   * Runs linters and test suites. If tests fail, the pipeline halts immediately.
2. **Build & Push Stage (`build-and-push` job):**
   * Executes *only* if the test stage passes successfully (`needs: test`).
   * Authenticates securely against Docker Hub.
   * Leverages `docker/build-push-action` along with GitHub Actions cache backend (`cache-from: type=gha`) for ultra-fast builds.
   * Tags the generated image with the specific Git commit SHA (e.g., `sha-a1b2c3d`) as well as `latest`.

### Setup Requirements (GitHub Secrets)
To make this pipeline functional, navigate to your GitHub Repository -> **Settings** -> **Secrets and variables** -> **Actions**, and add the following repository secrets:
* `DOCKER_USERNAME`: Your personal Docker Hub username.
* `DOCKER_TOKEN`: A Personal Access Token (PAT) generated from your Docker Hub Account Settings (do not use your raw password).

### Local Deployment with Generated Images
Once the GitHub Actions pipeline builds and pushes the images, you can update your root `docker-compose.yml` file to pull those verified images directly:

```yaml
version: '3.8'

services:
  backend:
    image: your-dockerhub-username/dream-vacation-backend:latest
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgres://user:password@postgres:5432/dream_db
    depends_on:
      - postgres

  frontend:
    image: your-dockerhub-username/dream-vacation-frontend:latest
    ports:
      - "80:80"
    depends_on:
      - backend

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: dream_db
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:

  ### Key Highlights of this Setup:
* **Path Filtering Optimization:** By using `paths:`, editing frontend code won't waste your build limits spinning up backend runners.
* **Docker Layer Caching (`gha`):** It utilizes GitHub Actions native caching for Docker layers. Subsequent builds will complete in seconds rather than minutes because layers that haven't changed will be pulled straight from the cache.
* **Git Commit Tagging:** Tagging with `type=sha,format=short` satisfies your requirement perfectly, enabling deterministic deployments where you know exactly which commit is currently running in your container environment.