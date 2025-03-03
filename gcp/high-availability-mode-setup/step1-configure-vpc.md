# Google Cloud VPC Configuration Guide

This guide will walk you through the process of setting up a GCP Virtual Private Cloud (VPC) using the GCP Console.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Configure GCP VPC & Subnets](#step-1-create-a-vpc)
3. [Configure Public Subnets](#step-2-configure-public-subnets)
4. [Conclusion](#conclusion)


## Prerequisites

Before you begin, ensure you have the following:
- A GCP account
- Access to the GCP Console
- IAM permissions to create VPC resources

## Configure GCP VPC & Subnets

### Step 1: Create a VPC

1. Log in to the [GCP Management Console](https://console.cloud.google.com/).

2. Navigate to the **VPC Dashboard** by searching for `VPC networks` in the GCP services search bar.
    
    ![Alt Text](/assets/gcp/search-vpc.png)

3. Click on the **Create VPC** button.

    ![Alt Text](/assets/gcp/create-vpc-network.png)

4. Fill in the following details for the VPC network:
   
    - **Name**: a string value composed of lowercase letters, numbers, and hyphens
    - **Description**: optional string value to add some details about the VPC network

    ![Alt Text](/assets/gcp/vpc-network-details.png)

5. Under the same VPC network configuration page configure the subnet.
    
    - **Name**: same naming rules as the VPC network above apply
    - **Description**: again, an optional field to provide more details
    - **Region**: select a region from the drop down menu
    - **IPv4 range**: configure an IP range for the subnet (example IPv4 range: `10.19.0.0/24`)
    - **Private Google Access**: if the GKE cluster will be created with private nodes, this must be enabled to connect the cluster with this subnet
    - **Flow logs**: set flow logs to `On` and then change the new `Aggregation interval` field to whatever frequency is preferred for logging

    ![Alt Text](/assets/gcp/subnet-configuration.png)

6. If both an external db and external Redis are configured in EIC, repeat step 5 to configure a second subnet.

    - To do so, first select `Add Subnet` underneath the already configured subnet from step 5:

    ![Alt Text](/assets/gcp/add_subnet.png)

    - Then repeat step 5, with the following notes:
        - **Region**: select the same region as the first subnet
        - **IPv4 range**: it is recommended to configure a different IP range to avoid endpoint reservation issues (example of a second IPv4 range: `10.161.0.0/28`)
        - **Private Google Access**: this can be left On/Off depending on the needs of the project and how the SQL/Redis instance is configured (in this documentation it will be left `Off`)

7. Scroll to the bottom of the screen and click on the `CREATE` button.
    
    ![Alt Text](/assets/gcp/create-vpc-btn.png)

8. The resulting VPC network should look similar to the image below:

    ![Alt Text](/assets/gcp/example-vpc-network.png)


### Conclusion

The GCP VPC network should now be successfully configured! [Next](./step2-configure-gke.md), a GKE Cluster can be attached to this VPC and configured accordingly.

For more information, refer to the [GCP VPC network documentation](https://cloud.google.com/vpc/docs/vpc).