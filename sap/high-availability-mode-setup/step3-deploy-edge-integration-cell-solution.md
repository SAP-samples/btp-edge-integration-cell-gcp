# Deploy SAP Edge Integration Cell on Google Kubernetes Engine (GKE) - Deploy Edge Integration Cell Solution

## Introduction

In the previous instruction, we've added the Edge Node, established the secure connection between the Edge Lifecycle Management and the Edge Node, and deployed Edge Lifecycle Management into your GKE cluster. Well done!

Now it is time for us to deploy the Edge Integration Cell solution into your GKE cluster. Upon successfully deploy the Edge Integration Cell solution, it will makes your GKE cluster as a real runtime location for running Integration Flow and APIs in the SAP Integration Suite.

In this instruction, we will walk you through the steps of deploy Edge Integration Cell solution

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Deploy the Edge Integration Cell Solution](#step-1-deploy-the-edge-integration-cell-solution)
- [Step 2. Configure keystore for Edge Integration Cell runtime](#step-2-configure-keystore-for-edge-integration-cell-runtime)
- [Step 3. Configure DNS mapping for Edge Integration Cell endpoint](#step-3-configure-dns-mapping-for-edge-integration-cell-endpoint)
- [Conclusion](#conclusion)


## Prerequisites
1. You've finish previous instruction [Deploy SAP Edge Integration Cell on Google Kubernetes Engine (GKE) - Add an Edge Node](/sap/high-availability-mode-setup/step2-add-edge-node.md), and have Edge Node available.

2. You've **finish all the previous GCP instructions**, and have below information handy.
    - **GKE Cluster**
        - Have Kubernetes configuration file of your GKE Cluster handy.
    - **Google Cloud SQL**
        - Have Google Cloud SQL for PostgreSQL database created.
        - Have Google Cloud SQL for PostgreSQL database connection endpoint point handy.
        - Have Google Cloud SQL for PostgreSQL database user name handy.
        - Have Google Cloud SQL for PostgreSQL database user password handy.
        - Have Google Cloud SQL for PostgreSQL database name handy.
        - Have Google Cloud SQL for PostgreSQL database SSL/TLS certificate file handy.
    - **Google Cloud Memorystore**
        - Have Google Cloud Memorystore for Redis data store created.
        - Have Google Cloud Memorystore for Redis data store connection endpoint point handy.
        - Have Google Cloud Memorystore for Redis data store user name handy.
        - Have Google Cloud Memorystore for Redis data store user password handy.
        - Have Google Cloud Memorystore for Redis data store SSL/TLS certificate file handy.

## Step 1. Deploy the Edge Integration Cell Solution

1. Open the Edge Lifecycle Management user interface. Select the Edge Node where you want to deploy the solution. Scroll down to the **Deployments** section, then click **Deploy Solution**.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-1.png)

2. In the **Solution Information** step, do following then click **Next Step**.

    - **Solution Details**
        - **Name:** Choose **Edge Integration Cell**.
        - **Version:** Choose **latest version**

    - **Solution Properties**
        - **Default Virtual Host:** Enter a **custom domain name** used for exposing Edge Integration Cell endpoints under the DNS provider used in the instruction [Domain and SSL Setup Guide for SAP Edge Integration Cell](/gcp/high-availability-mode-setup/step3-configure-domain-name-key-pair.md)
            - You DO NOT have to create this custom domain name record in your NDS provider, it will be created automatically during the solution deployment phase.
        - **Default Virtual Host Key Alias:** Enter the first part of your custom domain name.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-2.png)

3. In the **Dependencies** step, **leave everything as default**, then click **Next Step** directly.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-3.png)

4. In the **Edge Integration Cell Services** step, do the following, then click **Next Step**.

    - **Properties**
       
        - Check **Enable Multi AZ Support**.

        - Check **Enable External DB**.
            - **External DB Hostname:** Enter the Google Cloud SQL for PostgreSQL DB cluster **writer endpoint** that we created in the instruction [Google Cloud SQL for PostgreSQL Configuration Guide](/gcp/high-availability-mode-setup/step5-configure-rds.md).
            - **External DB Port:** Enter 5432 or the port number you gave when create the PostgreSQL DB cluster on Google Cloud SQL.
            - **External DB Username** Enter **postgres**
            - **External DB Password:** Enter the password of your PostgreSQL DB user.
            - **External DB TLS Root Certificate:** Upload the SSL/TLS certificates of your Google Cloud SQL for PostgreSQL DB cluster.

        - Check **Enable External Redis**
            - **External Redis Addresses:** Enter the Google Memorystore for Redis cluster endpoint, which we created in the instruction of [Google Memorystore with Redis Configuration Guide](/gcp/high-availability-mode-setup/step6-configure-redis.md)
            - **External Redis Mode:** Choose **cluster** or choose the mode of your Redis data store.
            - **External Redis Username:** Enter the redis user name that we create in the instruction of [Google Memorystore with Redis Configuration Guide](/gcp/high-availability-mode-setup/step6-configure-redis.md)
            - **External Redis Password:** Enter the redis user password that we create in the instruction of [Google Memorystore with Redis Configuration Guide](/gcp/high-availability-mode-setup/step6-configure-redis.md)
            - **External Redis TLS Certificate:** Upload the SSL/TLS certificate of your redis cluster.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-4.png)

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-5.png)

5. In the **Istio** step, do the following, then click **Next Step**.

    - **Properties**
        - Check **Client IP Preservation** check box.
        - **Load Balancer Provider:** Choose **Other**.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-6.png)

6. In the **Review** step, click Deploy button directly to trigger the Edge Integration Cell deployment.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-7.png)

7. The deployment of the solution starts with the deployment of the dependent solutions Istio and Edge Integration Cell Services. Once the deployment of independent solutions are complete, the deployment of Edge Integration Cell starts automatically.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-8.png)

8. Once all the deployment finished, you should see the screen as below.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-9.png)

## Step 2. Configure keystore for Edge Integration Cell runtime

A keystore is used to secure message exchange both at transport level and at message level. For Edge Integration Cell, since you can have more than one keystore, you can decide whether to create a new, add, remove, or delete an existing keystore.

1. Go back to the SAP Integration Suite home page. Go to **Monitor -> Integrations and APIs**, choose **Edge Integration Cell** runtime from the **Runtime** drop down, and then click **Keystore** card.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-10.png)

2. In the Manage Keystore screen, select **All** from **Runtime dropdown**. Then click Create button to create a new Keystore for Edge Integration Cell runtime.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-11.png)

3. Select the key store you just created, then choose **Add -> Key Pair** to upload the key pair file of your domain to the key store.

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-12.png)

4. In the Add Key Pair pop-up screen, do following then click **Add**.

    - **Alias:** Enter the **Default Virtual Host Key Alias** of your Edge Integration Cell solution deployment. You could find this value in your Edge Integration Cell deployment General tab.

    - **Files:** Upload the key pair file we created in the previous instruction [Domain and SSL Setup Guide for SAP Edge Integration Cell](/gcp/high-availability-mode-setup/step3-configure-domain-name-key-pair.md)

    - **Password:** Enter the **Export Password** you gave when you generate the key pair file in the previous instruction [Domain and SSL Setup Guide for SAP Edge Integration Cell](/gcp/high-availability-mode-setup/step3-configure-domain-name-key-pair.md)

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-13.png)

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-14.png)


## Step 3. Configure DNS mapping for Edge Integration Cell endpoint

See the guide for DNS mapping configuration [here](/gcp/high-availability-mode-setup/step3-configure-domain-name-key-pair.md).
    
Let's now verify the connectivity of the Edge Integration Cell endpoint. Go back to SAP Integration Suite home page. Go to **Operate**, choose the Edge Integration Cell node from the drop down. **Copy and paste the virtual host name of your Edge Integration Cell node in the browser**.

![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-18.png)

In your browser you should see the error message as below. This is an valid response and if your are seeing the error message below, that means your SAP Edge Integration Cell deployment has been completed and finished successfully!

    ```json
    {
        "error": {
            "code": "artifactNotFound",
            "message": "No API endpoint is registered with this host and path. Please ensure that the respective artifact is successfully deployed."
        }
    }
    ```

    ![Alt Text](/assets/sap/ha-mode/deploy-eic-solution-19.png)

## Conclusion

Deploying the SAP Edge Integration Cell on GKE involves a series of well-defined steps that integrate various GCP services to create a robust edge infrastructure. 

By following the steps outlined in this instruction, you have successfully set up the Edge Integration Cell with necessary dependencies like external databases, Redis clusters, and secure DNS configurations. 

This deployment not only ensures high availability through multi-AZ support but also provides secure communication channels through keystores and TLS certificates. 

With the Edge Integration Cell now operational, your enterprise is equipped to process and analyze data at the edge, reducing latency and improving the overall efficiency of your business operations. The final connectivity verification confirms that your setup is complete and ready for use, providing a solid foundation for extending SAP's capabilities to edge environments.

## References

- **SAP**
    - [Deploy the Edge Integration Cell Solution](https://help.sap.com/docs/integration-suite/sap-integration-suite/deploy-edge-integration-cell-solution)
    - [Interact with Keystores from Edge Integration Cell](https://help.sap.com/docs/integration-suite/sap-integration-suite/interact-with-keystores-from-edge-integration-cell?version=CLOUD)
    - [What is Keystore](https://help.sap.com/docs/integration-suite/sap-integration-suite/keystore?version=CLOUD&q=keystore)