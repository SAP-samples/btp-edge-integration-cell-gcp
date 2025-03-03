# Google Kubernetes Engine (GKE) Configuration Guide

This guide will walk you through setting up an Google Kubernetes Engine (GKE) cluster.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Create GKE Cluster](#step-1-create-gke-cluster)
3. [Enable NAT Gateway and Router](#step-2-enable-nat-gateway-and-router)
4. [Configure Communication with the GKE Cluster](#step-3-configure-communication-with-the-gke-cluster)

## Prerequisites

Before you begin, ensure you have the following:
- A GCP account
- Access to the GCP Console or `gcloud` CLI downloaded locally
- IAM permission to create and manage GKE clusters

## Step 1. Create GKE Cluster

1. Navigate to the Kubernetes Engine.

    ![Alt Text](/assets/gcp/gke-search.png)

2. Click on the `CREATE` button.

    ![Alt Text](/assets/gcp/create-gke.png)

3. After clicking on `CREATE` a pop-up will appear, select the standard option.

    ![Alt Text](/assets/gcp/standard-cluster-selection.png)

    - Note that technically this should not matter, but for the sake of configuring each detail by hand in this tutorial select the standard option

4. Now the creation page for a GKE cluster will appear. The first section covers `Cluster basics`, and the following details will need to be addressed:

    - **Name**: another string to name the resource
    - **Location type**: selecting `Regional` and then a region from the `Region` dropdown is acceptable for a simple EIC configuration. For high availability (recommended!), select `Zonal` and then a zone from the `Zone` dropdown (this should be in the same region as where the subnets configured, i.e. zone `us-west1-a` if the region of the subnet was `us-west1`). Additionally, check off `Specify default node locations` to increase the availability of nodes (select as many of the zones as desired).

    An example of the high availability configuration:

    ![Alt Text](/assets/gcp/cluster-basics-configuration.png)

5. Next, navigate to the `Configure node settings` page - on the panel to the left this is under NODE POOLS -> default-pool -> Nodes - and update the machine type from the default `e2-medium` to `e2-standard-8`.

    The node settings page:

    ![Alt Text](/assets/gcp/configure-node-settings-page.png)

    Machine type settings:

    ![Alt Text](/assets/gcp/machine-type-settings.png)

6. Head over to the Networking configuration page on the same panel - CLUSTER -> Networking - and configure the following settings:
    
    The configuration page should look like the following:

    ![Alt Text](/assets/gcp/networking-gke.png)

    - The following items require attention:
        - **Enable authorized networks**: this step is only required if the GKE cluster is to be private
            - Check the `Enable authorized networks`
            - Then under `Authorized networks` enter a `Name` and `Network` value
                - For the `Network` value, it is recommended to enter `0.0.0.0/0` to allow all connections, such that EIC can be bootstrapped onto the GKE cluster

        ![Alt Text](/assets/gcp/authorized-networks.png)

        - **Cluster networking**: the cluster is connected to the subnet at this step
            - Under `Network` select the name of the network created in the VPC network configuration step
            - Under `Node subnet` select the name of the subnet dedicated for the GKE cluster
                - If the GKE cluster has been set to private - see the following bullet - then ensure the subnet selected has `Private Google Access` enabled
            - Select `Enable Private nodes` to make the GKE cluster private

            ![Alt Text](/assets/gcp/gke-subnet-attachment.png)

7. After configuring all of these settings, click on `CREATE` at the bottom of the screen:

    ![Alt Text](/assets/gcp/create-gke-cluster-btn.png)


8. Here's an example of the configured cluster:
    
    ![Alt Text](/assets/gcp/example-cluster-post-config.png)

    And what it looks like within the VPC:

    ![Alt Text](/assets/gcp/cluster-connected-to-subnet.png)

## Step 2. Enable NAT Gateway and Router 

This step is only required if private nodes were enabled in the GKE cluster

- To create the router:
    ```
    gcloud compute routers create <router-name> \
        --network=<vpc-name> \
        --region=<region>
    ```
    - The `--network` flag should be the name of the configured VPC
    - The `--region` flag should match the region of the subnets in the VPC
    - An example:
    ![Alt Text](/assets/gcp/example-router.png)

- To create the NAT gateway:
    ```
    gcloud compute routers nats create <nat-gateway-name> \
        --router=<router-name> \
        --region=<region> \
        --auto-allocate-nat-external-ips \
        --nat-all-subnet-ip-ranges
    ```
    - The `--router` flag must match the name of the router created above
    - Again `--region` should align with the region of the router and subnets
    - An example:
    ![Alt Text](/assets/gcp/example-nat.png)

- Allowing outside connections to the VPC network through the gateway:
    ```
    gcloud compute firewall-rules create <FIREWALL_NAME> \
        --network <vpc-name> \
        --allow tcp,udp,icmp \
        --source-ranges 10.0.0.0/8
    ```
    - Once again, `--network` should match the name of the created VPC
    ![Alt Text](/assets/gcp/example-firewall-rule.png)



## Step 3. Configure Communication with the GKE Cluster

This section will detail how to acquire the **kubeconfig** file tied to the provisioned GKE cluster. The settings in this file enable the kubectl CLI to communicate with the cluster. The commands to acquire this file can be run locally; however, the steps below will detail how to run the commands within the GCP Cloud Shell, which makes life a little bit easier.

1. Navigate back to the `Kubernetes Engine` page and select the name of created cluster from the `OVERVIEW` tab.

    ![Alt Text](/assets/gcp/cluster-overview-page.png)

2. Click on `CONNECT` at the top of the screen.

    ![Alt Text](/assets/gcp/connect-to-cluster-btn.png)

3. On the resulting pop-up, click on `RUN IN CLOUD SHELL`.

    ![Alt Text](/assets/gcp/run-in-cloud-shell.png)

4. The GCP Cloud Shell should pop up. Before running the connect command, run the following command: `export KUBECONFIG=./config`.

5. Now run the connection command.

6. The connection command will have saved the config file in `./config`, so run `less ./config` to see the kubeconfig file.

7. Copy the contents of the resulting command and save it to a local yaml file.


## Conclusion

The GKE cluster is now set up with three node groups. At this point the configuration for EIC bootstrapping is complete.

## References

- **SAP**
    - [SAP Edge Integration Cell - Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
    - [SAP Edge Integration Cell - Prepare Your Kubernetes Cluster](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
    - [SAP Edge Integration Cell - Prepare for Deployment on GKE](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-for-deployment-on-google-kubernetes-engine-gke)

- **GCP**
    - [Getting started with GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)
    - [Getting started with Google Cloud Shell](https://cloud.google.com/shell/docs)
