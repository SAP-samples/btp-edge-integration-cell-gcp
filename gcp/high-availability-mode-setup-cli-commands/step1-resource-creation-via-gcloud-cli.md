# Overview of GCP Resources

This guide provides an overview of GCP resource configuration via the `gcloud` CLI. It assumes the following [prerequisites](step0-prerequisite.md) have been met. It goes in order of what resources to configure, starting with the [VPC](#vpc) and ending with [PostgresSQL](#postgresql). Each section provides the CLI commands to configure each resource, along with some notes detailing things to keep in mind.

Some recommendations to make the executing the following commands easier:
 - Set `<project_id>` as an environment variable. Then each `<project_id>` that needs to be filled out below can be replaced with the corresponding environment variable.
 - The same applies to `<region>`, as this flag appears frequently in the CLI configuration commands below. 
 - While creating these resources, make sure to keep track of their names, as some resource names will be used in the creation and configuration of other resources. Feel free to choice any string value to replace the name templates in the command below.

This guide also creates a private service connect (PSC) resource unique to Redis and PostgreSQL. Technically, only one PSC is needed to configure the line of sight between the VPC and each resource, but creating one PSC per resource prevents IP allocation issues.

## Table of Contents
1. [Setting up the VPC](#vpc)
    - [VPC Creation](#vpc-creation)
    - [Subnet Creation](#subnet-creation)
2. [Creating a GKE Cluster](#gke)
    - [Cluster Creation](#cluster-creation) 
    - [Router and NAT Gateway Creation](#router-and-nat-gateway-creation)
    - [Configure DNS Name](#configure-dns-name)
    - [Obtain Kube Config](#obtain-kube-config)
3. [Provisioning a Redis Instance](#redis)
    - [Setting up PSC](#setting-up-psc)
    - [Creating a Redis Instance](#creating-redis-instance)
    - [Redis EIC Details](#redis-eic-details)
4. [Spinning up a PostgreSQL Instance](#postgresql)
    - [Deploy PostgreSQL Instance](#deploy-postgresql-instance)
    - [Enable Private Service Connect](#enable-private-service-connect)
    - [PostgreSQL EIC Details](#postgresql-eic-details)

## VPC

We begin by creating the virtual private cloud and its subnets.

### VPC Creation

Virtual private cloud creation:
```
gcloud compute networks create <vpc_network_name> \
    --project=<project_id> \
    --subnet-mode=custom \
    --mtu=1460 \
    --bgp-routing-mode=regional \
    --bgp-best-path-selection-mode=legacy
```

### Subnet Creation

Now we need to configure the appropriate subnets.

Subnet creation for PostgreSQL:
```
gcloud compute networks subnets create <subnet_name> \
    --project=<project_id> \
    --range=<postgres_private_ip_range>/28 \
    --stack-type=IPV4_ONLY \
    --network=<vpc-network-name> \
    --region=<region> \
    --enable-flow-logs \
    --logging-aggregation-interval=interval-5-min \
    --logging-flow-sampling=0.5 \
    --logging-metadata=include-all
```

Subnet creation for Redis/GKE:
```
gcloud compute networks subnets create <subnet_name> \
    --project=<project_id> \
    --range=<redis_private_ip_range> \
    --stack-type=IPV4_ONLY \
    --network=<vpc-network-name> \
    --region=<region> \
    --enable-private-ip-google-access \
    --enable-flow-logs \
    --logging-aggregation-interval=interval-5-min \
    --logging-flow-sampling=0.5 \
    --logging-metadata=include-all
```

A few notes to keep in mind about configuring the VPC and it's subnets:
 1. Define the same `<region>` for each subnet and keep this consistent across the other resources provisioned. If a resource uses zones instead of regions, configure the resource's zone in that region (i.e. `us-west1-a` for the zone if the region is `us-west1`).
 2. An example value for the IP range of the PostgreSQL subnet: `10.161.0.0/28`.
 3. An example value for the IP range of the Redis/GKE subnet: `10.19.0.0/24`.

If only one PSC is created, be careful with IP range allocation for PostgreSQL and Redis. It is possible to accidentally configure ranges such that the first service configured takes up all of the private IP ranges, preventing the second service from connecting to its respective subnetwork.

## GKE

Next, we configure the kubernetes cluster and connect it to the VPC above. Furthermore, we enable EIC to bootstrap to the cluster.

### Cluster Creation

Kubernetes cluster creation:
```
gcloud beta container \
    --project <project_id> \
    clusters create <cluster_name> \
    --zone <zone> \
    --tier "standard" \
    --no-enable-basic-auth \
    --cluster-version <valid_cluster_version> \
    --release-channel "regular" \
    --machine-type "e2-standard-8" \
    --image-type "COS_CONTAINERD" \
    --disk-type "pd-balanced" \
    --disk-size "100" \
    --metadata disable-legacy-endpoints=true \
    --scopes "https://www.googleapis.com/auth/devstorage.read_only,\
https://www.googleapis.com/auth/logging.write,\
https://www.googleapis.com/auth/monitoring,\
https://www.googleapis.com/auth/servicecontrol,\
https://www.googleapis.com/auth/service.management.readonly,\
https://www.googleapis.com/auth/trace.append" \
    --max-pods-per-node "110" \
    --num-nodes "3" \
    --logging=SYSTEM,WORKLOAD \
    --monitoring=SYSTEM,STORAGE,POD,DEPLOYMENT,STATEFULSET,DAEMONSET,HPA,CADVISOR,KUBELET \
    --enable-private-nodes \
    --enable-ip-alias \
    --network "projects/<project_id>/global/networks/<network_name>" \
    --subnetwork "projects/<project_id>/regions/<region_of_vpc>/subnetworks/<subnet_name>" \
    --no-enable-intra-node-visibility \
    --default-max-pods-per-node "110" \
    --enable-autoscaling \
    --min-nodes "0" \
    --max-nodes "3" \
    --location-policy "BALANCED" \
    --enable-ip-access \
    --security-posture=standard \
    --workload-vulnerability-scanning=disabled \
    --enable-master-authorized-networks \
    --master-authorized-networks 0.0.0.0/0 \
    --no-enable-google-cloud-access \
    --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver \
    --enable-autoupgrade \
    --enable-autorepair \
    --max-surge-upgrade 1 \
    --max-unavailable-upgrade 0 \
    --binauthz-evaluation-mode=DISABLED \
    --enable-managed-prometheus \
    --enable-shielded-nodes \
    --node-locations "<zone>-a,<zone>-b,<zone>-c"
```

A few things to note:
- For the `--network` and `--subnetwork` flags, ensure that `<project_id>`, `<network_name>`, and `<subnet_name>` are replaced accordingly. The latter two should match the creation commands in the [VPC](#vpc) section above, particularly the `<subnet_name>` should match the name of the subnet dedicated to Redis/GKE.
- Enable zone instead of region to have redundancy across three node-locations (for high availability).
- `<cluster_version>` needs to be a valid (and ideally newer) version of kubernetes
    - use the following command to get valid versions: `gcloud container get-server-config --zone <zone>`
- The default machine type does not have enough RAM to run EIC; we are using the more powerful `"e2-standard-8"`. This can be seen with the following flag: `--machine-type "e2-standard-8"`.
- For bootstrapping EIC to the cluster, master authorized networks needs to be enabled and the IP address needs to accept all connections:
```
--enable-master-authorized-networks \
--master-authorized-networks 0.0.0.0/0 \
```
- Note that allowing any network to connect to the cluster can be removed after the initial bootstrap step, with the following command:
```
gcloud container clusters update <cluster_name> \
  --zone <zone> \
  --no-enable-master-authorized-networks
```

### Router and NAT Gateway Creation

After creating the GKE cluster, if nodes are private - this is enabled with the flag `--enable-private-nodes` - then to create a line of sight for EIC a NAT gateway and router must be created to expose the GKE cluster. This can be done with the following steps:

Router creation:
```
gcloud compute routers create <router-name> \
      --network=<vpc-name> \
      --region=<region>
```

NAT gateway creation:
```
gcloud compute routers nats create <nat-gateway-name> \
    --router=<router-name> \
    --region=<region> \
    --auto-allocate-nat-external-ips \
    --nat-all-subnet-ip-ranges
```

Allowing outside connections:
```
gcloud compute firewall-rules create <FIREWALL_NAME> --network <vpc-name> --allow tcp,udp,icmp --source-ranges 10.0.0.0/8
```

After this is completed, bootstrapping with the ELM can begin.



### Configure DNS Name

After EIC has been bootstrapped onto the GKE cluster, DNS should be configured for the Istio load balancer so that the rest of the EIC deployment may occur.

#### Prerequisites

The following steps detail how to create an A record type for the DNS. If a CNAME configuration is preferred, please refer to this [Google Cloud guide](https://cloud.google.com/identity/docs/add-cname). Additionally, a managed zone is needed in Google Cloud DNS, and for more information on that, please refer to [this guide](https://cloud.google.com/dns/docs/zones) on zone creation.

#### Commands 

First, make sure a connection is established to the GKE cluster:
```
gcloud container clusters get-credentials <cluster_name> \
    --zone <zone> \
    --project <project_id>
```

Make sure to replace `<cluster_name>` and `<zone>` with the name of the created cluster and its zone, respectively. Additionally, replace `<project_id>` with the project ID of the Google Cloud workspace.

Here is the command to fetch the IP address of the Istio load balancer:
```
echo $(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

The returned IP address will be used below.

Here is the command to create the A record:
```
gcloud dns --project=<project_id> record-sets create <full_dns_name> \
    --zone=<managed_zone> \
    --type="A" \
    --ttl="300" \
    --rrdatas=<ip_address>
```

`<managed_zone>` should be replaced by the Cloud DNS Zone you would like to use. `<full_dns_name>` should be the alias and the zone combined: "alias_name.managed_zone_name.". Make sure to include the period at the end of the string! Finally, the `<ip_address>` should be the value coming from the Istio command previously executed.

### Obtain Kube Config

For EIC bootstrapping, the Kube config file is needed. To acquire this, run the following commands in the Google Cloud Shell.

Update Kube config file location:
```
export KUBECONFIG=./config
```

Connect to the created cluster:
```
gcloud container clusters get-credentials <cluster_name> \
    --zone <zone> \
    --project <project_id>
```

Run `less ./config` and copy the contents of the file. This will be uploaded to EIC during bootstrapping as the config file of the cluster.


## Redis

### Setting up PSC

Reserving an IP address for the PSC:
```
gcloud compute addresses create <redis_psc_name> \
    --global \
    --purpose=VPC_PEERING \
    --prefix-length=<prefix_length> \
    --description=<description> \
    --network=<vpc_network_name>
```

The `--description` flag is optional.

The `--prefix_length` can be set to any desired value, but generally this was set to `20` or `24` in testing.

Connecting the PSC to the VPC:
```
gcloud services vpc-peerings connect \
    --service=servicenetworking.googleapis.com \
    --ranges=<redis_psc_name> \
    --network=<vpc_network_name>
```

Confirming the connection:
```
gcloud services vpc-peerings list \
    --network=<vpc_network_name>
```

This command should output the following string: `servicenetworking.googleapis.com`.

### Creating Redis Instance

The Redis instance can be created after creating the PSC for Redis and confirming its successful configuration.

Provision the Redis instance:
```
gcloud redis instances create <redis_instance_name> \
    --region=<region> \
    --zone=<zone> \
    --tier=BASIC \
    --redis-version=<redis_version> \
    --connect-mode=PRIVATE_SERVICE_ACCESS \
    --network=<vpc_network_name> \
    --enable-auth \
    --transit-encryption-mode=SERVER_AUTHENTICATION \
    --replica-count=0
```

Again, keep the `<region>` value consistent with the value selected for the creation of the VPC.

The `<zone>` value should be the `<region>` value followed by `-a`, `-b`, or `-c`. For example, if the configured region is `us-west1`, then the zone can be any of `us-west1-a`, `us-west1-b`, or `us-west1-c`.

The `--redis-version` flag can be set to `redis_7_x`.

Replica count must be set to `0` for a `BASIC` tier Redis instance.

### Redis EIC Details

Now that the Redis instance has been created and connected to the VPC, its configuration details can be copied over to the EIC deployment configuration panel.

`External Redis Addresses` can be found by heading over to the newly created Redis instance in the GCP console. After clicking on the Redis instance, select `Connections` from the panel on the left. Then, under the `Connections` header copy the `Primary endpoint` value. This is the `External Redis Addresses`. 

`External Redis Mode` is `standalone`.

`External Redis Password` can be found in the GCP console. Navigate to the newly created Redis instance and click on the `Security` tab (on the left panel). Scroll down to the `AUTH string` header and copy the string listed there. This is the `External Redis Password`.  

To obtain the `server.pem` file for the `External Redis TLS Certificate`, stay on the `Security` tab. Scroll down to the `TLS Certificate Authority` header and press the `DOWNLOAD` button to obtain the `server.pem` file. 

## PostgreSQL

This section details how to create the PostgreSQL instance and how to create the line of sight to the VPC via Private Service Connect.

Any value in `<>` will need to be replaced. Here's a list of values that will be changed across commands:
 - `<project_id>`: this is the account's project ID - here's how to [locate the project ID](https://support.google.com/googleapi/answer/7014113)
 - `<region>`: a valid GCP region - this should align with the VPC or subnet, details on which are specified after each command
 - `<postgres_instance_name>`: the name of the PostgreSQL DB Instance to create
 - `<psc_name>`: a string to name the psc
 - `<subnet_name>`: the name of a subnet in the created VPC network
 - `<ip_address>`: a valid IP address
 - `<new_password>`: string that will serve as a password 
 - `<psc_endpoint_name>`: another string, in this case identifying the endpoint of the psc
 - `<vpc_network_name>`: the name of the created VPC network
 - `<dns_name>`: a string to identify the DNS managed zone for the instance 
 - `<dns_record_string>`: a string to identify the DNS record for the instance

Additional notes for flags may proceed each command.

### Deploy PostgreSQL Instance

PostgresSQL instance creation:
```
gcloud sql instances create <postgres_instance_name> \
    --project=<project_id> \
    --region=<region> \
    --enable-private-service-connect \
    --allowed-psc-projects=<project_id> \
    --availability-type=ZONAL \
    --no-assign-ip \
    --cpu=2 \
    --memory=7680MB \
    --edition=ENTERPRISE \
    --database-version=POSTGRES_15
```

As mentioned before, ensure that the `<region>` value is aligned with the subnet dedicated for this PostgresSQL instance. 

To update the password for the default postgres user:
```
gcloud sql users set-password postgres \
    --instance=<postgres_instance_name> \
    --password=<new_password>
```

### Enable Private Service Connect

Now it's time to create a PSC to enable the line of sight from the PostgreSQL instance to the VPC.

Reserve an internal IP address for the PSC endpoint in the VPC:
```
gcloud compute addresses create <psc_name> \
    --project=<project_id> \
    --region=<region> \
    --subnet=<subnet_name> \
    --addresses=<ip_address>
```

Note that the `<ip_address>` above should be derived from the subnet created in the VPC. So, following the example IP range for the PostgreSQL subnet, an example value could be `10.161.0.10`. Additionally, ensure that `<subnet_name>` is replaced with the corresponding name of the Redis subnet. 

Verify that the internal IP address is reserved properly:
```
gcloud compute addresses list <psc_name> \
    --project=<project_id>
```

This command should output the following:
```
NAME: <psc_name>
ADDRESS/RANGE: 10.161.0.10  # this is following the example value - this should output whatever value `<ip_address>` was replaced with
TYPE: INTERNAL
PURPOSE: GCE_ENDPOINT
NETWORK:
REGION: <region>
SUBNET: <subnet_name>
STATUS: RESERVED
```

Next, grab the service attachment URI from the PostgreSQL instance for the created PSC:
```
gcloud sql instances describe <postgres_instance_name> \
    --project=<project_id> \
    --format="value(pscServiceAttachmentLink)"
```

This command should output a service attachment string in this format:
```
projects/<project_id>/regions/<region>/serviceAttachments/<attachment_string>
```

Use this service attachment string to create the PSC endpoint:
```
gcloud compute forwarding-rules create <psc_endpoint_name> \
    --address=<psc_name> \
    --project=<project_id> \
    --region=<region> \
    --network=<vpc_network_name> \
    --target-service-attachment=<service_attachment_string> \
    --allow-psc-global-access
```

The endpoint can be verified via the following command:
```
gcloud compute forwarding-rules describe <psc_endpoint_name> \
    --project=<project_id> \
    --region=<region> \
    --format="value(pscConnectionStatus)"
```

If the endpoint is successfully configured, this command should output `ACCEPTED`.

Next, create a DNS managed zone for the PostgreSQL instance:
```
gcloud dns managed-zones create <dns_name> \
    --project=<project_id> \
    --description=<description> \
    --dns-name=<region>.sql.goog. \
    --networks=<vpc_network_name> \
    --visibility=private
```

The `--description` flag is optional. Keep the `<dns_name>` value saved for future commands.

Then grab the DNS record:
```
gcloud sql instances describe <postgres_instance_name> --project=<project_id> --format="value(dnsName)"
```

This will output a string, which is the DNS record. Use this DNS record to create the DNS managed zone:
```
gcloud dns record-sets create <dns_record_string> \
    --project=<project_id> \
    --type=A \
    --rrdatas=<ip_address_used_for_postgres_instance_creation> \
    --zone=<dns_name>
```

Reuse the `<dns_name>` value here from the DNS managed zone create command.

### PostgreSQL EIC Details

At this point, the external PostgreSQL instance should be fully configured and all of the information for the EIC deployment can be copied over.

`External DB Hostname` field is the DNS name created that ends in `.sql.goog.`.

`External DB Port` is `5432`.

`External DB Name` is `postgres` by default. This can be changed if another database is provisioned.

`External DB Username` is also `postgres` by default.

`External DB Password` was set in the `gcloud sql users set-password postgres ` command.

To grab the certificate for the EIC deployment configuration panel, head to the PostgreSQL instance and navigate to the `Connections` tab (on the left side of the screen). Under `Connections`, click on `SECURITY` from the four header tabs. In this tab, under the `Manage SSL mode` section set `SSL mode` to `Allow only SSL connections`. Wait until the instance updates. Then, under the same `SECURITY` tab, under the `Manage server CA certificates` header click on `DOWNLOAD CERTIFICATES` to obtain the necessary `server.pem` file to upload to the EIC deployment configuration panel for the `External DB TLS Root Certificate` field.