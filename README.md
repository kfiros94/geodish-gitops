🍽️ geodish-gitops

This repository manages the GitOps deployments for the geodish application and its supporting infrastructure components using Argo CD and Helm.

It adheres to the "App of Apps" pattern, where a single root application manages the deployment and lifecycle of all sub-components.

🚀 Application Overview (geodish-app)

The core application, named geodish-app, is a Python-based Flask REST API. Its deployment is managed by a dedicated Helm chart that includes:

    Deployment: Runs 2 replicas of the application.

        Image Source: 893692751288.dkr.ecr.ap-south-1.amazonaws.com/geodish-app:58.

        Configuration: Environment variables are set via a ConfigMap (FLASK_ENV: "production", PORT: "5000").

        Database Connection: The MongoDB URI is securely sourced from a Kubernetes Secret named mongodb-secret using the key connectionString.

    Service: A ClusterIP service exposing port 5000.

    Ingress: Enabled by default.

        Host: kfir-cowsay.ddns.net.

        TLS/HTTPS: Uses the letsencrypt-prod ClusterIssuer and manages TLS via the geodish-tls Secret.

    Database Seeding: A Kubernetes Job (seed-job) runs after the application is up to initialize the database by sending a POST request to the /seed endpoint.

⚙️ GitOps Flow: The "App of Apps" Pattern

This repository deploys the entire stack using the Argo CD App of Apps model:

    Root Application: The geodish-root-app is defined in argocd/applications/root-app.yaml. It tracks the helm-charts/app-of-apps directory.

    App Aggregation: The app-of-apps chart is responsible for generating multiple child Argo CD Application resources using the applications.yaml template.

    Component Sync: Each generated child Application tracks one of the component charts (e.g., geodish-app, mongodb, logging, monitoring), ensuring all parts of the ecosystem are synced automatically by Argo CD.

    Sync Policy: The root application is configured for automated sync with prune: true and selfHeal: true.

📂 Repository Structure

Directory / File	Purpose
argocd/applications/root-app.yaml	The top-level Argo CD application that targets the app-of-apps chart.
helm-charts/app-of-apps/templates/applications.yaml	Template used to generate all child Argo CD applications (e.g., for geodish-app, mongodb).
helm-charts/geodish-app/	The core application's Helm chart definition.
helm-charts/geodish-app/templates/deployment.yaml	Defines the Kubernetes Deployment for the Flask application.
helm-charts/geodish-app/templates/seed-job.yaml	Defines the database initialization job that uses curl to call the /seed endpoint.
helm-charts/geodish-app/values.yaml	Default configuration for the geodish-app deployment.
helm-charts/logging/, mongodb/, monitoring/	Directories for supporting infrastructure charts.

🛠️ Getting Started

Prerequisites

    A running Kubernetes Cluster.

    Argo CD installed in the cluster (e.g., in the argocd namespace).

    A pre-existing Kubernetes Secret named mongodb-secret containing the connection string key must be present in the target namespace.

Initial Deployment

To bootstrap the entire system, apply the root application resource directly to your cluster, targeting the namespace where Argo CD is installed.

    Clone the repository:
    Bash

git clone https://github.com/kfiros94/geodish-gitops.git
cd geodish-gitops

Apply the Root Application: This command registers the top-level application with Argo CD, which will then automatically deploy the rest of the stack.
Bash

    kubectl apply -f argocd/applications/root-app.yaml -n argocd

    Monitor Status: Use the Argo CD UI or CLI to watch the synchronization status of geodish-root-app and verify that all child applications are deploying correctly.

📚 Customization

Configuration for each deployed component is managed via its respective values.yaml file.
Configuration Area	File to Modify	Default Value Example
Application Image Tag	helm-charts/geodish-app/values.yaml	tag: "58"
Ingress Hostname	helm-charts/geodish-app/values.yaml	kfir-cowsay.ddns.net
Replica Count	helm-charts/geodish-app/values.yaml	replicaCount: 2
Resource Limits	helm-charts/geodish-app/values.yaml	cpu: 500m, memory: 512Mi
Child Applications/Namespaces	helm-charts/app-of-apps/values.yaml (indirectly via template)	Defines which path goes to which namespace