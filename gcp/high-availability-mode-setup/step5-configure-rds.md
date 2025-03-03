# Google Cloud SQL for PostgreSQL Configuration Guide

## Introduction

SAP Edge Integration Cell can be configured with external services for managing persistence and policies. In this case, a relational database system is may be enabled by SAP Edge Integration Cell.

For the DEV / UAT environments, an internal PostgreSQL database will be deployed as built-in Edge Integration Cell components.

For the PROD environment, the built-in PostgreSQL is not highly available and scalable. It is recommended to configure an external PostgreSQL DB cluster for the PROD env.

On the Google side, a Cloud SQL PostgreSQL DB cluster can be created.

The requirements below on the external PostgreSQL database must be meet:

- PostgreSQL version **12** and **15 (preferred)** are currently supported as external database by SAP Edge Integration Cell.

- **Minimum CPU / Memory requirements: 1 CPU / 2 GiB**

- PostgreSQL server must have TLS/SSL enabled.

- Server certificate must use a Subject Alternative Name (SAN) extension (type DNS Name) to be used as server identity.

- PostgreSQL connectivity and user information is required during solution deployment.

- Required user privileges: GRANT ALL on schema

For more information about the requirements on PostgreSQL database, please referring to the following SAP official documents

- [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839) 
- [PostgreSQL Database Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)

This instruction, will go through the steps of creating a PostgreSQL Database on Google's Cloud SQL for SAP Edge Integration Cell.


## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create an Google Cloud PostgreSQL Database Instance](#step-1-create-an-google-cloud-postgresql-database-instance)
- [Step 2. Create a new DB](#step-2-create-a-postgressql-db)
- [Step 3. Create a DB user](#step-3-create-a-db-user)
- [Step 4. Obtain EIC Parameters](#step-4-obtain-eic-parameters)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

Before starting, ensure the following prerequisites are met:
- Completing the [Google VPC Network Configuration Guide](./step1-configure-vpc.md)
- Completing instruction for [Google Kubernetes Engine (GKE)](./step2-configure-gke.md)
- To connect to the Cloud SQL instance from local, download pgAdmin:
    - Download: https://www.pgadmin.org/download/
- IAM permissions to create and manage Cloud SQL instances.

## Step 1. Create an Google Cloud PostgreSQL Database Instance

**Note:** While this section does create a Google Cloud PostgreSQL Database Instance, the automatic Private Service Connect created by GCP in the UI does not currently work for EIC. It is instead recommended to use these [supplemental CLI commands](./step5-supplemental-rds-commands.md) instead of following this step. The remainder of the guide can be followed normally.

1. Navigate to the Cloud SQL Instances page

    ![Alt Text](/assets/gcp/search-for-cloud-sql.png)

2. Select `CREATE INSTANCE` near the top of the screen

    ![Alt Text](/assets/gcp/create-sql-instance-start.png)

3. Select `PostgreSQL` rom the database engine options

    ![Alt Text](/assets/gcp/sql-db-engine-selection.png)

4. From the resulting configuration page, select the following options:

    - **Cloud SQL version:** select `Enterprise`
    - **Edition preset:** choose between `Production`, `Development`, and `Sandbox` as needed (compute size decreases from left to right across these options)

    ![Alt Text](/assets/gcp/sql-config-1.png)

    - **Database version:** select `PostgreSQL 12` or `PostgreSQL 15` from the dropdown
    - **Instance ID:** type in a string to identify the instance
    - **Password:** set the password for the default `postgres` user

    ![Alt Text](/assets/gcp/sql-config-2.png)

    - **Region:** select the same region as the configured subnets, from the `Region` dropdown
    - **Zonal availability:** for high availability select `Multiple Zones (Highly available)` and the two prefered zones for the `Primary zone` and `Secondary zone`

    ![Alt Text](/assets/gcp/sql-config-3.png)

    Under `Configuration Options`, navigate to `Connections`
    - Select `Private IP`
    - Select the configured VPC network from the dropdown
    - Select `Enable private path`

    ![Alt Text](/assets/gcp/sql-config-4.png)

5. Select `CREATE INSTANCE` at the bottom of the configuration page

    ![Alt Text](/assets/gcp/create-instance-btn-sql-after-config.png)


## Step 2. Create a PostgresSQL DB

1. Navigate to `Databases`

2. Click on `CREATE DATABASE`

    ![Alt Text](/assets/gcp/create-db-btn.png)

3. In the resulting pop-up, enter a `Database Name` and select `CREATE`

    ![Alt Text](/assets/gcp/create-db-name.png)

## Step 3. Create a DB User

1. Navigate to `Users`

2. Click on `ADD USER ACCOUNT`

    ![Alt Text](/assets/gcp/create-user-account-btn.png)

3. Enter a new user name and password and the press `ADD`

    ![Alt Text](/assets/gcp/create-user-creds.png)

## Step 4. Obtain EIC Parameters

1. Navigate to `Connections` -> `Security` -> `Manage SSL Mode`

2. Select `Allow Only SSL Connections`

    ![Alt Text](/assets/gcp/sql-ssl-cert.png)

3. Select `DOWNLOAD CERTIFICATES` to obtain the SSL/TLS `server-ca.pem` certificate file for EIC deployment

    ![Alt Text](/assets/gcp/sql-ssl-cert-download.png)

## Conclusion

Now a Google Cloud SQL for PostgreSQL DB instance should be successfully configured in the GCP account. 

Please keep below the following information handy for EIC deployment:

- **Instance connection endpoint**
- **User name**
- **User password**
- **Database name**
- **SSL/TLS connection certificate**

## References

- **SAP**
    - [PostgreSQL Database Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
    - [Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
    - [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)

- **GCP**
    - [Cloud SQL docs](https://cloud.google.com/sql/docs)