# GCP Memorystore for Redis

## Introduction

SAP Edge Integration Cell requires external services for managing persistence and policies. In this case, an in-memory data store used for caching is required by SAP Edge Integration Cell.

For the DEV / UAT environments, Redis services will be deployed as built-in Edge Integration Cell components.

For the PROD environment, built-in Redis data store is not highly available and scalable. It is highly encouraged to configure an external Redis cluster for the PROD env.

On the GCP side, Memorystore for Redis can be used as external Redis data store for the SAP Edge Integration Cell. The requirements below on the external Redis cluster of your choice must be meet:

- **Redis** versions **6.x**, **7.x** are currently supported as external data store by SAP Edge Integration Cell.
- **Minimum CPU / Memory requirements: 1 CPU / 1 GiB**
- Redis server must have **TLS/SSL enabled**.
- Server certificate must use a Subject Alternative Name (SAN) extension (type DNS Name) to be used as server identity.
- Redis connectivity and user (**with full permission**) information is required during solution deployment.

For more information about the requirements on Redis, please refer to the following SAP official documents
- [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)
- [Redis Data Store Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster#redis-data-store-requirements)

The rest of this guide will detail how to configure the Redis instance on GCP's Memorystore, meeting the requirements above.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create a Redis Instance](#step-1-create-a-redis-instance)
- [Step 2. Obtain EIC Parameters](#step-2-obtain-eic-parameters)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

Before starting, ensure the following criteria is met:

- The [VPC network from step one](./step1-configure-vpc.md) has been created.
- IAM permissions are enabled for created a Redis instance on GCP Memorystore.

## Step 1. Create a Redis Instance

1. Navigate to `Memorystore for Redis` via the search bar
    
    ![Alt Text](/assets/gcp/search-for-redis-memory-store.png)

2. Select `CREATE INSTANCE`
    
    ![Alt Text](/assets/gcp/create-redis-instance-start.png)

3. Configure the following parameters:

    - **Instance ID:** a name to identify the instance
    - **Tier Selection:** for testing purposes select `Basic` and for production deployments select `Standard`
    - **Capacity:** choose a size based on need, for a development setup 5 GB 
    - **Choose region and zonal availability:** select the same region as configured for each subnet from the `Region` dropdown, and select a zone from the `Zone` dropdown if the tier was set to `Basic`
    - **Read Replicas:** configure as needed, but two read replicas are recommended - note that read replicas are disabled for a `Basic` tier
    - **Set up connection:** from the `Network` dropdown, select the VPC created in step one

    An example of what this might look like so far:
    ![Alt Text](/assets/gcp/redis-partial-example.png)

    - **Additional Configurations:**
        - **Connections:**
            - Under connections, select `Private service access`
            - On the resulting pop-up, select `SET UP CONNECTION`

            ![Alt Text](/assets/gcp/setup-private-serivce-connect-redis.png)

            - A sidebar on the right hand side of the screen should appear
                - Select `Use an automatically allocated IP range`
                - Then press `CONTINUE`

                ![Alt Text](/assets/gcp/psc-redis-step1.png)

                - Finally, press `CREATE CONNECTION`
                
        - **Security:**
            - Select both `Enable Auth` and `Enable in-transit encryption`

            ![Alt Text](/assets/gcp/redis-security-config.png)

        - **Maintenance:**
            - For production environments, select a day and time from the `Day` and `Time` dropdowns that would be best for maintenance

        - **Configuration:**
            - Select a version from the `Version` dropdown (7.0 is recommended)
            - If desired, enable the option to take snapshots of the Redis instance

4. Click on `CREATE INSTANCE` at the bottom of the configuration page

    ![Alt Text](/assets/gcp/create-redis-instance-btn.png)

5. Here's an example of a configured Redis instance:

    ![Alt Text](/assets/gcp/redis-cluster-deployed.png)

## Step 2. Obtain EIC Parameters

1. Head to the newly created Redis instance

2. Navigate to the `Connections` page via the panel of the left hand side and copy the `Primary endpoint` value on this page

    ![Alt Text](/assets/gcp/redis-connection-endpoint.png)

3. Navigate to the `Security` page via the panel on the left hand side and copy the `AUTH string` value

    ![Alt Text](/assets/gcp/redis-auth-string.png)

4. On the same page, scroll down to the `TLS Certificate Authority` section and click on `DOWNLOAD` to obtain the `server.pem` file for the Redis instance (this is the SSL/TLS certificate)

    ![Alt Text](/assets/gcp/redis-cert.png)

## Conclusion

The Redis instance is now successfully created and configured! Please keep the following credentials below handy for EIC deployment configuration:

- **Redis Cluster Connection Endpoint**
- **Redis Cluster Auth String** (this will be the password for the db)
- **SSL/TLS Certificate of Redis Cluster** (the server.pem file)

## References

- **SAP**
    - [Redis Data Store Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
    - [Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
    - [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)

- **GCP**
    - [Memorystore for Redis docs](https://cloud.google.com/memorystore/docs/redis)