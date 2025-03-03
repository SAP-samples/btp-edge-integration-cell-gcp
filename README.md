# Deploy SAP Integration Suite - Edge Integration Cell on Google Cloud Platform (GCP)

SAP Edge Integration Cell is a hybrid integration runtime offered as part of SAP Integration Suite, which enables you to manage APIs and run integration scenarios within your private landscape.

This repository provides comprehensive instructions for deploying SAP Edge Integration Cell on Google Cloud Platform (GCP). By following the guidelines provided, you can configure the necessary GCP resources and deploy SAP Edge Integration Cell.

![Alt Text](/assets/gcp/eic-gcp-ard.png)


## Requirements

To ensure a successful and resilient deployment of SAP Edge Integration Cell in a production environment on GCP, the following infrastructure components are required:

- **SAP Prerequisites**
    - SAP BTP Global Account & Subaccount.
    - SAP Integration Suite (Standard Edition or Premium Edition) Service Entitlement & Subscription.
    - (Optional) SAP Integration Suite - Edge Integration Cell service plan entitlement.

- **External Prerequisites**

    - (**Mandatory**) [Google Virtual Private Cloud (VPC)](https://cloud.google.com/vpc/docs/vpc)

        - At least 3 Availability Zone with a Private subnet.

    - (**Mandatory**) Kubernetes cluster on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)

        - **Kubernetes** **1.26**, **1.27**, **1.28**, **1.29** are currently supported by Edge Integration Cell.

        - **Minimum 16 CPU, 64 GiB memory, and 204GiB persistent volumes are required for each Kubernetes node**.
        
        - The Kubernetes cluster must have Role-Based Access Control (RBAC) enabled to ensure secure and controlled access to resources.

        - 3 Kubernetes worker node are required, and these nodes must match the Kubernetes API server's version to maintain compatibility.

        - The cluster must support dynamic provisioning of persistent volumes with `ReadWriteOnce` and `ReadWriteMany` access modes.

        - A kubeconfig with cluster admin privileges is required for initializing the edge node, enabling CRUD operations on namespaces and CRDs.

    - (**Mandatory**) A PostgreSQL database on [Google Cloud SQL](https://cloud.google.com/sql/docs/introduction)

        - **PostgreSQL** **12** and **15 (preferred)** are currently supported by Edge Integration Cell.

        - **Minimum 1 CPU, 2GiB memory are required for the PostgreSQL database server**.

        - The PostgreSQL server must have TLS/SSL enabled to secure data transmission.

        - The server should provide a single endpoint (host and port pair) for connections.

    - (**Mandatory**) A Redis data store on [Google Memorystore](https://cloud.google.com/memorystore)

        - **Redis** **6.x** and **7.x** are currently supported by Edge Integration Cell.

        - **Minimum 1 CPU, 1 GiB memory are required for the Redis server**.
        
        - Redis server must have TLS/SSL enabled.

For more information, please refer to [SAP Notes 3247839](https://me.sap.com/notes/3247839) 

### Required Infrastructure Sizing Guidelines

Understand the key components of Edge Integration Cell, the factors that influence its performance, and the methods to calculate the size for the hardware resources required.

For detailed sizing guidelines and up-to-date infomation on versioning, please refer to [Sizing Guide for Edge Integration Cell](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)

## Deploy SAP Edge Integration Cell on on GCP
    
> **⚠️ Disclaimer:** 
> The networking setup, database, and datastore configuration illustrated in this repository are currently intended as a starting point for developers. Please adhere to your organization's specific guidelines on security and network configuration.

To get started with SAP Edge Integration Cell in your GCP landscape, simply follow the below steps:

**Step-by-Step Instructions:**

1. GCP Infrastructure Setup

    - [Pre-requisites for GCP Setup](/gcp/high-availability-mode-setup/step0-prerequisite.md)
        
        Ensure that all pre-requisites are met, including SAP BTP account setup, service entitlements, and GCP environment preparation.

    - [Configuring VPC on GCP](/gcp/high-availability-mode-setup/step1-configure-vpc.md)

        Set up a Virtual Private Cloud (VPC) on GCP, with the necessary availability zones, subnets and security configurations to support your Edge Integration Cell deployment.

    - [Kubernetes Cluster Setup on GKE](/gcp/high-availability-mode-setup/step2-configure-gke.md)

        Create and configure an GKE cluster that meets the requirements for Edge Integration Cell deployment. Additionally, setup a Google Cloud NAT Gateway and Router.

    
    - [Configure DNS for GKE](/gcp/high-availability-mode-setup/step3-configure-domain-name-key-pair.md)

    - [Configure PostgreSQL database on Google Cloud SQL](/gcp/high-availability-mode-setup/step5-configure-rds.md)

        Set up a PostgreSQL database on Google Cloud SQL, ensuring it meets the performance and security requirements for Edge Integration Cell.

    - [Configure Redis on Google Memorystore](/gcp/high-availability-mode-setup/step6-configure-redis.md)

        Deploy a Redis instance on Google Memorystore, which will serve as the data store for asynchronous messaging within the Edge Integration Cell.

2. **SAP Edge Integration Cell Setup**

    - [Activate Edge Integration Cell on SAP Integration Suite](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md)

        Enable the Edge Integration Cell service within your SAP Integration Suite, preparing it for deployment.

    - [Add Edge Node and Bootstrap GKE Cluster](/sap/high-availability-mode-setup/step2-add-edge-node.md)

        Register your Kubernetes cluster as an edge node within the SAP system and bootstrap the necessary configurations.

    - [Deploy Edge Integration Cell Solution](/sap/high-availability-mode-setup/step3-deploy-edge-integration-cell-solution.md)

        Deploy the SAP Edge Integration Cell solution on your edge node, completing the integration between your GCP infrastructure and SAP Integration Suite.

## Conclusion

By following these steps, you will successfully deploy the SAP Edge Integration Cell in a highly available and scalable environment on GCP, fully integrated with your SAP Integration Suite. This setup ensures that your data remains secure within your private landscape while leveraging the powerful capabilities of SAP's hybrid integration model.

## Known Issues

No known issues.

## How to obtain support
[Create an issue](https://github.com/SAP-samples/<repository-name>/issues) in this repository if you find a bug or have questions about the content.
 
For additional support, [ask a question in SAP Community](https://answers.sap.com/questions/ask.html).

## Contributing
If you wish to contribute code, offer fixes or improvements, please send a pull request. Due to legal reasons, contributors will be asked to accept a DCO when they create the first pull request to this project. This happens in an automated fashion during the submission process. SAP uses [the standard DCO text of the Linux Foundation](https://developercertificate.org/).

## License
Copyright (c) 2024 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSE) file.
