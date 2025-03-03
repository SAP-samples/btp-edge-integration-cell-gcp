# Domain and SSL Setup Guide for SAP Edge Integration Cell

## Introduction

When deploying SAP Edge Integration Cell, ensuring seamless access and optimal traffic management is crucial. To achieve this, a Load Balancer will be configured to expose the Edge Integration Cell endpoint, effectively distributing traffic across the Kubernetes (K8s) nodes and services. A key aspect of this setup is the registration of a custom hostname, which will be associated with the Load Balancer in the Domain Name System (DNS).

This guide, will cover the process of creating and configuring a domain name specifically for the SAP Edge Integration Cell, utilizing Google Cloud DNS. This configuration will enable the secure and efficient exposure of the Edge Integration Cell endpoints.

The following steps detail how to create an A record type for the DNS. If a CNAME configuration is preferred, please refer to this [Google Cloud guide](https://cloud.google.com/identity/docs/add-cname).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Obtain Istio Load Balancer IP Address](#step-1-obtain-istio-load-balancer-ip-address)
- [Step 2. Create A Record](#step-2-create-a-record)
- [Conclusion](#conclusion)

## Prerequisites

Before starting, ensure the following points are met:
 - A Cloud DNS Zone is configured
    - For more information, refer to [this guide](https://cloud.google.com/dns/docs/zones) on Zone creation
 - The previous guides have been followed and a GKE cluster has been created
 - The bootstrapping process for the Edge Integration Cell has already been completed
    - The Istio load balancer will not be deployed on the GKE cluster until this has occurred, and the IP address of the Istio load balancer is required for DNS configuration

## Step 1. Obtain Istio Load Balancer IP Address

1. Navigate to Google Cloud Shell

2. Make sure a connection is established with the provisioned GKE cluster:

The command for connecting:
```
gcloud container clusters get-credentials <cluster_name> \
    --zone <zone> \
    --project <project_id>
```

Make sure to replace `<cluster_name>` and `<zone>` with the name of the created cluster and its zone, respectively. Additionally, replace `<project_id>` with the project ID of the Google Cloud workspace.

3. Run the following command to get the IP address of the Istio load balancer:

```
echo $(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

Save the IP address that is returned by the command.

## Step 2. Create A Record

1. Navigate to Google Cloud's Network Services
    
    ![Alt Text](/assets/gcp/navigte-to-network-services.png)

2. Click on the `Cloud DNS` (see panel on the left) and then select the configured DNS Zone

3. Select `ADD STANDARD`

    ![Alt Text](/assets/gcp/add-standard-dns.png)

4. Enter the following details:

    - **DNS name:** enter an alias to create the DNS name
    - **IPv4 Address 1:** enter the IP address that was saved from the first step of this guide

    ![Alt Text](/assets/gcp/record-set-config.png)

5. Select `CREATE`

    ![Alt Text](/assets/gcp/create-btn-record-set.png)

## Conclusion

A custom domain should be successfully configured for SAP Edge Integration Cell deployment. Please keep the DNS name handy for EIC configuration.