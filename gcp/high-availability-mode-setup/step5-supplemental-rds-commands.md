# Google Cloud SQL Instance CLI Commands

This guide serves as an addendum to the first step of the [Google Cloud SQL Instance setup guide](./step5-configure-rds.md#step-1-create-an-google-cloud-postgresql-database-instance). Currently, creating a Cloud SQL PostgreSQL instance will automatically configure a Private Service Connect connection to the VPC network. Unfortunately, this automatic connection does not work with EIC.

Follow these Google Cloud CLI commands to properly configure a PostgreSQL instance with a working Private Service Connect connection.

## Table of Contents

- [PostgreSQL gcloud Commands](#postgresql-gcloud-commands)
- [Deploy PostgreSQL Instance](#deploy-postgresql-instance)
- [Enable Private Service Connect](#enable-private-service-connect)
- [EIC Parameters to Save](#eic-parameters-to-save)

## PostgreSQL gcloud Commands

It is recommended to run each of the commands below in Google Cloud Shell. Although, it is possible to run them locally via the gcloud CLI.

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

Replace `<postgres_instance_name>` with a string to name the instance.

Ensure that the `<region>` value is aligned with the subnet dedicated for this PostgresSQL instance. Replace `<project_id>` with the account's project ID. Here's how to [locate the project ID](https://support.google.com/googleapi/answer/7014113).

This next command can be skipped, as the Google Console guide covers how to manage users.

To update the password for the default postgres user:
```
gcloud sql users set-password postgres \
    --instance=<postgres_instance_name> \
    --password=<new_password>
```

Replace `<postgres_instance_name>` and `<new_password>` with the name of the instance (from the command above) and a new password value, respectively.

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

Make sure to properly substitute for the values in this command. This command should output the following:
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

Reuse the `<dns_name>` value here from the DNS managed zone create command. The `<ip_address_used_for_postgres_instance_creation>` is the same IP address used in this command `gcloud compute addresses create <psc_name>`.

### EIC Parameters to Save

The only EIC parameter to save from this guide, that will not be covered in the main Google Cloud SQL PostgreSQL Instance creation guide, is the `connection endpoint`. This value is the output of this command: `gcloud sql instances describe <postgres_instance_name> --project=<project_id> --format="value(dnsName)"`.