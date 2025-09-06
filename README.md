# Laravel Docker & Kubernetes CI/CD Template

This repository serves as a comprehensive template for deploying Laravel applications using Docker and Kubernetes, automated via GitHub Actions. It includes configurations for building Docker images, pushing to GitHub Container Registry, and deploying to `test`, `staging`, and `production` environments with MySQL database and phpMyAdmin.

## Features:

*   **Dockerization:** `Dockerfile` for Laravel application.
*   **Kubernetes Manifests:** `k8s/` directory with base and overlay configurations for:
    *   Laravel Deployment, Service, HPA (Horizontal Pod Autoscaler)
    *   MySQL StatefulSet, Service, PVC, ConfigMap, and Secret
    *   Laravel Database Migration Job
    *   NGINX Ingress for application and phpMyAdmin
    *   phpMyAdmin Deployment, Service, and Ingress
*   **GitHub Actions CI/CD Pipeline (`.github/workflows/deploy.yml`):**
    *   Automated build, test (PHPUnit), and vulnerability scanning (Snyk, Trivy).
    *   Deployment to `develop` branch (test environment).
    *   Deployment to `main` branch (staging environment).
    *   Manual deployment to `production` environment via `workflow_dispatch`.
    *   Dynamic image naming based on GitHub repository.

## How to Use This as a Template for a New Laravel Project:

To adapt this setup for a new Laravel application, follow these steps:

### Step 1: Prepare Your New Laravel Project

1.  **Create a New GitHub Repository:** Create an empty GitHub repository for your new Laravel application (e.g., `your-username/new-laravel-app`).
2.  **Initialize Laravel Project:** Set up your new Laravel application locally.
3.  **Copy Template Files:** Copy the entire `.github/` directory and the `k8s/` directory from this template repository into the root of your new Laravel project.

    ```bash
    # Assuming you are in your new Laravel project's root directory
    cp -r /path/to/this/template/repository/.github .
    cp -r /path/to/this/template/repository/k8s .
    ```

### Step 2: Customize Kubernetes Configurations (`k8s/` directory)

You'll need to adjust specific values within the `k8s/` directory to match your new project's details.

1.  **Update Ingress Hostnames:**
    Modify the `host:` field in the following files to reflect your new application's domain.
    *   `k8s/overlays/test/ingress.yaml` (e.g., `test.your-new-app.com`)
    *   `k8s/overlays/staging/ingress.yaml` (e.g., `staging.your-new-app.com`)
    *   `k8s/overlays/production/ingress.yaml` (e.g., `prod.your-new-app.com`)
    *   `k8s/base/phpmyadmin/phpmyadmin-ingress.yaml` (e.g., `phpmyadmin.your-new-app.com`)

2.  **Configure Database Details:**
    *   **`k8s/base/mysql/mysql-configmap.yaml`:**
        *   Update `MYSQL_DATABASE` to your desired database name for the new project (e.g., `your_new_app_db`).
    *   **`k8s/base/mysql/mysql-secret.yaml`:**
        *   **Choose a strong MySQL root password** for your new project.
        *   **Base64 encode** this password: `echo -n "YOUR_NEW_MYSQL_ROOT_PASSWORD" | base64`
        *   Replace the `password:` value in this file with your new base64 encoded password.

3.  **Review Other Kubernetes Files:**
    *   **`k8s/base/mysql/mysql-pv-claim.yaml`:** Adjust `storage` size if needed.
    *   **`k8s/base/mysql/mysql-statefulset.yaml`:** `image: mysql:8.0` can be changed if you need a different MySQL version.
    *   **`k8s/base/deployment.yaml`:** `image: ghcr.io/ThisIsDeeraj/docker-demo:latest` is a placeholder; the CI/CD pipeline will replace this with your new project's image.
    *   **`k8s/base/migration-job.yaml`:** The `image:` here is also a placeholder and will be replaced by the CI/CD pipeline.

### Step 3: Configure GitHub Repository Settings

For your CI/CD pipeline to function correctly in your new repository, you must set up specific GitHub Secrets and Environments.

1.  **`KUBE_CONFIG` Secret:**
    *   This secret holds the `kubeconfig` content for your Kubernetes cluster (e.g., MicroK8s).
    *   On your server where MicroK8s is running, get the content: `microk8s config`
    *   Go to your new GitHub repository's **Settings > Secrets and variables > Actions**.
    *   Click **New repository secret**, name it `KUBE_CONFIG`, and paste the entire output from `microk8s config` into the "Value" field.

2.  **`SNYK_TOKEN` Secret (Optional, if using Snyk):**
    *   If you plan to use Snyk for vulnerability scanning, you'll need your Snyk API token.
    *   Go to your new GitHub repository's **Settings > Secrets and variables > Actions**.
    *   Click **New repository secret**, name it `SNYK_TOKEN`, and paste your Snyk API token into the "Value" field.

3.  **`production` Environment:**
    *   The `deploy_production` job requires a GitHub Environment named `production`.
    *   Go to your new GitHub repository's **Settings > Environments**.
    *   Click **New environment**, name it `production`, and click **Configure environment**. You can add protection rules if desired.

### Step 4: Push Your Code and Trigger the Pipeline

1.  **Commit Your Changes:**
    ```bash
    git add .
    git commit -m "feat: Initial setup for CI/CD pipeline"
    ```

2.  **Push to GitHub:**
    ```bash
    git push origin main # Or 'git push origin develop' if you prefer to start there
    ```
    This push will automatically trigger your new CI/CD pipeline. Monitor its progress in your repository's "Actions" tab.

### Step 5: Access Your Application and phpMyAdmin

Once the pipeline completes successfully:

1.  **Update Your Local `hosts` File:**
    On your local machine (Mac/Linux: `/etc/hosts`, Windows: `C:\Windows\System32\drivers\etc\hosts`), add entries mapping your new application's hostnames to your server's public IP address.
    ```
    <YOUR_SERVER_PUBLIC_IP> test.your-new-app.com
    <YOUR_SERVER_PUBLIC_IP> staging.your-new-app.com
    <YOUR_SERVER_PUBLIC_IP> prod.your-new-app.com
    <YOUR_SERVER_PUBLIC_IP> phpmyadmin.your-new-app.com
    ```

2.  **Flush DNS Cache:**
    ```bash
    sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder # macOS
    ```

3.  **Access in Browser:**
    *   Your application: `http://staging.your-new-app.com/` (or `http://test.your-new-app.com/`, `http://prod.your-new-app.com/`)
    *   phpMyAdmin: `http://phpmyadmin.your-new-app.com/` (Login with `root` and your chosen MySQL root password).

This template provides a solid foundation for your Laravel deployments on Kubernetes!
