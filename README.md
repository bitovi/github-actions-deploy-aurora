# Deploy Aurora DB Cluster (Postgres/MySQL)
`bitovi/github-actions-deploy-aurora` deploys an Aurora cluster with any amount of instances, with the option for a proxy.

This action uses our new GitHub Actions Commons repository, a library that contains multiple Terraform modules, allowing us to condense all of our tools in one repo, hence continuous improvements are made to it. 
![alt](https://bitovi-gha-pixel-tracker-deployment-main.bitovi-sandbox.com/pixel/C_iypLByw8-MNrgzYYMC_)
## Action Summary
This action creates an Aurora Cluster, with the option to add even a proxy. Could be a Postgres or MySQL. 

If you would like to deploy a backend app/service, check out our other actions:
| Action | Purpose |
| ------ | ------- |
| [Deploy Docker to EC2](https://github.com/marketplace/actions/deploy-docker-to-aws-ec2) | Deploys a repo with a Dockerized application to a virtual machine (EC2) on AWS |
| [Deploy React to GitHub Pages](https://github.com/marketplace/actions/deploy-react-to-github-pages) | Builds and deploys a React application to GitHub Pages. |
| [Deploy static site to AWS (S3/CDN/R53)](https://github.com/marketplace/actions/deploy-static-site-to-aws-s3-cdn-r53) | Hosts a static site in AWS S3 with CloudFront |
<br/>

**And more!**, check our [list of actions in the GitHub marketplace](https://github.com/marketplace?category=&type=actions&verification=&query=bitovi)

# Need help or have questions?
This project is supported by [Bitovi, A DevOps consultancy](https://www.bitovi.com/services/devops-consulting).

You can **get help or ask questions** on our:

- [Discord Community](https://discord.gg/J7ejFsZnJ4Z)

Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/services/devops-consulting).

# Basic Use - Postgres

For basic usage, create `.github/workflows/deploy.yaml` with the following to build on push.
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy-aurora
      uses: bitovi/github-actions-deploy-aurora@v0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

```

# Basic MySQL
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy-aurora
      uses: bitovi/github-actions-deploy-aurora@v0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

        aws_aurora_engine: aurora-mysql
```

# Advanced use
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - id: deploy
        uses: bitovi/github-actions-deploy-aurora@v0.1.0
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_default_region: us-east-1

          aws_additional_tags: '{\"some\":\"extra\",\"tag\":\"added\"}'

          tf_state_bucket_destroy: true

          aws_aurora_engine: aurora-mysql
          aws_aurora_proxy: true
          aws_aurora_cluster_apply_immediately: true
          aws_aurora_database_name: some-db-name
          aws_aurora_master_username: master
          aws_aurora_ingress_allow_all: true
          aws_aurora_subnets: subnet-0000000000000,subnet-0000000000000
          aws_aurora_db_instance_class: db.r6g.large
          aws_vpc_id: vpc-0000000000000
          aws_resource_identifier: replaced-this-from
          tf_state_bucket: bitovi-resources
          tf_state_file_name_append: aurora-dev-db
```

# Multi-AZ cluster
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - id: deploy
        uses: bitovi/github-actions-deploy-aurora@v0.1.0
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_default_region: us-east-1

          tf_state_bucket_destroy: true
          aws_aurora_db_instances_count: 0
  
          aws_aurora_engine: postgres
          aws_aurora_availability_zones: us-east-1a,us-east-1b,us-east-1c
          aws_aurora_cluster_db_instance_class: db.m5d.large
          aws_aurora_storage_type: io1
          aws_aurora_storage_iops: 1000
          aws_aurora_allocated_storage: 100
  
          aws_aurora_proxy: true
```

### Inputs
1. [AWS Specific](#aws-specific)
1. [Action default inputs](#action-default-inputs)
1. [Aurora Inputs](#aurora-inputs)
1. [Aurora Proxy Inputs](#aurora-proxy-inputs)
1. [VPC Inputs](#vpc-inputs)

### Outputs
1. [Aurora Outputs](#aurora-outputs)

The following inputs can be used as `step.with` keys
<br/>

#### **AWS Specific**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_access_key_id` | String | AWS access key ID |
| `aws_secret_access_key` | String | AWS secret access key |
| `aws_session_token` | String | AWS session token |
| `aws_default_region` | String | AWS default region. Defaults to `us-east-1` |
| `aws_resource_identifier` | String | Set to override the AWS resource identifier for the deployment. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. |
| `aws_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to all provisioned resources.|
<br/>

#### **Action default inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `tf_stack_destroy` | Boolean  | Set to `true` to destroy the stack. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. Defaults to `tf-state-aws`. |
| `tf_state_file_name_append` | String | Appends a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. (Can co-exist with `tf_state_file_name`) |
| `tf_state_bucket` | String | AWS S3 bucket name to use for Terraform state. See [note](#s3-buckets-naming) | 
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. Default is `false`. |
| `bitops_code_only` | Boolean | If `true`, will run only the generation phase of BitOps, where the Terraform and Ansible code is built. |
| `bitops_code_store` | Boolean | Store BitOps generated code as a GitHub artifact. |
<br/>


#### **Aurora Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_aurora_enable` | Boolean | Toggles deployment of an Aurora database. Defaults to `true`. |
| `aws_aurora_proxy` | Boolean | Aurora DB Proxy Toggle. Defaults to `false`. |
| `aws_aurora_cluster_name` | String | The name of the cluster. Defaults to `aws_resource_identifier` if none set. |
| `aws_aurora_engine` | String | The database engine to use. Defaults to `aurora-postgresql`. |
| `aws_aurora_engine_version` | String | The DB version of the engine to use. Will default to one of the latest selected by AWS. More information [Postgres](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.20180305.html) or [MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraMySQLReleaseNotes/Welcome.html)|
| `aws_aurora_engine_mode` | String | Database engine mode. Could be global, multimaster, parallelquey, provisioned, serverless. |
| `aws_aurora_availability_zones` | String | Comma separated list of zones to deploy DB to. If none, will automatically set this. | 
| `aws_aurora_cluster_apply_immediately` | Boolean | Apply changes immediately to the cluster. If not, will be done in next maintenance window. Defaults to `false`. |
| **Storage** |||
| `aws_aurora_allocated_storage` | String | Amount of storage in gigabytes. Required for multi-az cluster. |
| `aws_aurora_storage_encrypted` | Boolean | Toggles whether the DB cluster is encrypted. Defaults to `true`. |
| `aws_aurora_kms_key_id` | String | KMS Key ID to use with the cluster encrypted storage. |
| `aws_aurora_storage_type` | String | Define type of storage to use. Required for multi-az cluster. |
| `aws_aurora_storage_iops` | String | iops for storage. Required for multi-az cluster. | 
| **Cluster details** |||
| `aws_aurora_database_name` | String | The name of the database. will be created if it does not exist. Defaults to `aurora`. |
| `aws_aurora_master_username` | String | Master username. Defaults to `aurora`. |
| `aws_aurora_database_group_family` | String | The family of the DB parameter group. See [MySQL Reference](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/AuroraMySQL.Reference.html) or [Postgres Reference](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/AuroraPostgreSQL.Reference.html). Defaults automatically set for MySQL(`aurora-mysql8.0`) and Postgres (`aurora-postgresql15`). |
| `aws_aurora_iam_auth_enabled` | Boolean | Toggles IAM Authentication. Defaults to `false`. |
| `aws_aurora_iam_roles` | String | Define the ARN list of allowed roles. |
| `aws_aurora_cluster_db_instance_class` | String | To create a Multi-AZ RDS cluster, you must additionally specify the engine, storage_type, allocated_storage, iops and aws_aurora_db_cluster_instance_class attributes. |
| **Networking** |||
| `aws_aurora_security_group_name` | String | Name of the security group to use for postgres. Defaults to `SG for {aws_resource_identifier} - Aurora`.| 
| `aws_aurora_allowed_security_groups` | String | Extra names of the security groups to access Aurora. Accepts comma separated list of. |
| `aws_aurora_ingress_allow_all` | Boolean | Allow access from 0.0.0.0/0 in the same VPC. Defaults to `true`. |
| `aws_aurora_subnets` | String | Subnet ids to use for postgres. Accepts comma separated list of. |
| `aws_aurora_database_port` | String | Database port. Defaults to `5432`. |
| **Backup & maint** |||
| `aws_aurora_cloudwatch_enable` | Boolean | Toggles cloudwatch. Defaults to `true`. |
| `aws_aurora_cloudwatch_log_type` | String | Comma separated list of log types to include in cloudwatch. If none defined, will use [postgresql] or [audit,error,general,slowquery]. Based on the db engine. |
| `aws_aurora_cloudwatch_retention_days` | String | Days to store cloudwatch logs. Defaults to `7`. |
| `aws_aurora_backtrack_window` | String | Target backtrack window, in seconds. Only available for aurora and aurora-mysql engines currently. 0 to disable. Defaults to `0`. |
| `aws_aurora_backup_retention_period` | String | Days to retain backups for. Defaults to `5`. |
| `aws_aurora_backup_window` | String | Daily time range during which the backups happen. |
| `aws_aurora_maintenance_window` | String | Maintenance window. |
| `aws_aurora_database_final_snapshot` | String | Set the name to generate a snapshot of the database before deletion. |
| `aws_aurora_deletion_protection` | Boolean | Protects the cluster from deletion. Defaults to `false`. **This won't prevent db instances to be deleted.** To disable it, you'll have to go through the AWS Console. |
| `aws_aurora_delete_auto_backups` | Boolean | Specifies whether to remove automated backups immediately after the DB cluster is deleted. Default is `true`. |
| `aws_aurora_restore_snapshot_id` | String | Restore an initial snapshot of the DB if specified. |
| `aws_aurora_restore_to_point_in_time` | map{String} | Restore database to a point in time. Will require a map of strings. Like `{"restore_to_time"="W","restore_type"="X","source_cluster_identifier"="Y", "use_latest_restorable_time"="Z"}`. Default `{}`. |
| `aws_aurora_snapshot_name` | String | Takes a snapshot of the DB. This is treated as one resource, meaning only one can be created, even if name changes.|
| `aws_aurora_snapshot_overwrite` | Boolean | Takes a snapshot of the DB deleteing the previous snapshot. Defaults to `false`. |
| **DB Instance** |||
| `aws_aurora_db_instances_count` | String | Amount of instances to create. Defaults to `1`. |
| `aws_aurora_db_instance_class` | String | Database instance size. Defaults to `db.r6g.large`. |
| `aws_aurora_db_apply_immediately` | String | Specifies whether any modifications are applied immediately, or during the next maintenance window. Defaults to `false`. |
| `aws_aurora_db_ca_cert_identifier` | String | Certificate to use with the database. Defaults to `rds-ca-ecc384-g1`. |
| `aws_aurora_db_maintenance_window` | String | Maintenance window. |
| `aws_aurora_db_publicly_accessible` | Boolean | Make database publicly accessible. Defaults to `false`. | 
| `aws_aurora_additional_tags` | JSON | A JSON object of additional tags that will be included on created resources. Example: `{"key1": "value1", "key2": "value2"}`. |
<br/>

#### **Aurora Proxy Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_db_proxy_name` | String | Name of the database proxy.  Defaults to `aws_resource_identifier` |
| `aws_db_proxy_client_password_auth_type` | String | Overrides auth type. Using `MYSQL_NATIVE_PASSWORD` or `POSTGRES_SCRAM_SHA_256` depending on the database engine. |
| `aws_db_proxy_tls` | Boolean | Make TLS a requirement for connections. Defaults to `true`.|
| `aws_db_proxy_security_group_name` | String | Name for the proxy security group. Defaults to `aws_resource_identifier`. |
| `aws_db_proxy_database_security_group_allow` | Boolean | If true, will add an incoming rule from every security group associated with the DB. |
| `aws_db_proxy_allowed_security_group` | String | Comma separated list for extra allowed security groups. |
| `aws_db_proxy_allow_all_incoming` | Boolean | Allow all incoming traffic to the DB Proxy (0.0.0.0/0 rule). Keep in mind that the proxy is only available from the internal network except manually exposed. | 
| `aws_db_proxy_cloudwatch_enable` | Boolean | Toggle Cloudwatch logs. Will be stored in `/aws/rds/proxy/rds_proxy.name`. |
| `aws_db_proxy_cloudwatch_retention_days` | String | Number of days to retain cloudwatch logs. Defaults to `14`. |
| `aws_db_proxy_additional_tags` | JSON | Add additional tags to the ter added to aurora provisioned resources.|
<br/>

#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created. Defaults to `false`. |
| `aws_vpc_name` | String | Define a name for the VPC. Defaults to `VPC for ${aws_resource_identifier}`. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`|
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If no input, no private subnet will be created. Defaults to `<none>`. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region+<random>` value. If a list is defined, the first zone will be the one used for the EC2 instance. |
| `aws_vpc_id` | String | **Existing** AWS VPC ID to use. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | **Existing** AWS VPC Subnet ID. If none provided, will pick one. (Ideal when there's only one). |
| `aws_vpc_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to vpc provisioned resources.|
<br/>

#### **Aurora Outputs**
| Name             | Description                        |
|------------------|------------------------------------|
| `aurora_db_endpoint` | Aurora Endpoint. |
| `aurora_db_secret_details_name` | AWS Secret name containing db credentials. |
| `aurora_db_sg_id` | SG ID for the Aurora instance. |
| `aurora_proxy_endpoint` | Database proxy endpoint. |
| `aurora_proxy_secret_name` | AWS Secret name containing proxy credentials. |
| `aurora_proxy_sg_id` | SG ID for the Aurora Proxy instance. |

<br/>

## Contributing
We would love for you to contribute to [`bitovi/github-actions-deploy-aurora`](hhttps://github.com/bitovi/github-actions-deploy-aurora).   [Issues](https://github.com/bitovi/github-actions-deploy-aurora/issues) and [Pull Requests](https://github.com/bitovi/github-actions-deploy-aurora/pulls) are welcome!

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/bitovi/github-actions-deploy-aurora/blob/main/LICENSE).

# Provided by Bitovi
[Bitovi](https://www.bitovi.com/) is a proud supporter of Open Source software.

# We want to hear from you.
Come chat with us about open source in our Bitovi community [Discord](https://discord.gg/J7ejFsZnJ4Z)!
