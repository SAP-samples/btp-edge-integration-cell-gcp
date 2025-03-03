# Pre-requisites

This guide will walk you through the prerequisites and setup process of configure required resources on GCP for the SAP Edge Integration Cell deployment

Before setting up your resources on GCP, ensure you have completed the following prerequisites:
1. **GCP Account**
   - An active GCP account is required. If you don't have one, [get started here](https://cloud.google.com/cloud-console).

2. **Google Cloud CLI**
   - Install the Google Cloud CLI. Follow the instructions [here](https://cloud.google.com/sdk/docs/install).
   - This step is optional as Google Cloud CLI commands can be run from [Google Cloud Shell](https://cloud.google.com/shell/docs).

3. **kubectl**
   - Install `kubectl`, the Kubernetes command-line tool. Instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

4. **Required IAM permissions**
    - To configure these resources permissions to work with GKE IAM roles, service linked roles, a VPC, and related resources will be needed. 
    - Make sure to assign the following IAM roles to the user that will be provisioning the GCP resources:
        - `Compute Network Admin`: for managing VPCs, subnets, routes, firewalls, VPNs, etc.
        - `Kuberetes Engine Admin`: for managing GKE clusters, node pools, and workloads.
        - `Service Account User`: to bind service accounts to GKE workloads.
        - `Cloud Memorystore Redis Admin`: allows for the creation and management of Redis instances, including scaling and updates (optional - only required if configuring an external Redis instance in EIC).
        - `Cloud SQL Admin`: full control over Cloud SQL instances (optional - only required if configuring an external db Postgres instance in EIC).
        - `Compute Network Admin`: for configuring private IPs/VPC peering connections (required for creating private service connect endpoints)
    - Note that most of the roles above fall under `Editor`, but this is not recommended for production environments due to security concerns