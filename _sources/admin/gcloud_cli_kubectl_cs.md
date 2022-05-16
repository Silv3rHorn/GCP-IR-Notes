# gcloud CLI & kubectl Cheatsheet
```{admonition} Assumption
Default project is assumed to have been [configured](#defaults-set) for gcloud CLI. Hence, `--project` option is not specified in most of the listed commands.
```
```{warning}
Permissions required for each listed command had been provided on a best effort basis. There are some commands with only the required **write** permissions provided and had been indicated as such.
```

## Setting Up

### Install Components
[Reference](https://cloud.google.com/sdk/gcloud/reference/components/install)
```
# install kubectl
gcloud components install kubectl
```

### Configuration Profile
[Reference](https://cloud.google.com/sdk/gcloud/reference/config/configurations)
```
# list profiles
gcloud config configurations list

# create new profile
gcloud config configurations create <profile_name>

# configure new profile
see Authenticate & Defaults below
 
# activate profile
gcloud config configurations activate <profile_name>
```

### Authenticate
[Reference](https://cloud.google.com/sdk/gcloud/reference/auth)
```
# authenticate as service account
gcloud auth activate-service-account --key-file <json_key_file>

# authenticate as user account
gcloud auth login
```

### Defaults - Set
[Reference](https://cloud.google.com/sdk/gcloud/reference/config/set)
```
# set default project
gcloud config set core/project <project_name>

# set default compute region and zone
gcloud config set compute/region <region>
gcloud config set compute/zone <zone>

# set default service account to impersonate
gcloud config set auth/impersonate_service_account <service_account>
```

### Defaults - Unset
[Reference](https://cloud.google.com/sdk/gcloud/reference/config/unset)
```
# unset default project
gcloud config unset core/project

# unset default compute region and zone
gcloud config unset compute/region
gcloud config unset compute/zone

# unset default service account to impersonate
gcloud config unset auth/impersonate_service_account
```

### Impersonate Service Account
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts)
```
# impersonate using user account
gcloud iam service-accounts add-iam-policy-binding <target_acct> \
    --member='user:<user_acct>' \
    --role='roles/iam.serviceAccountTokenCreator'

# impersonate using service account
gcloud iam service-accounts add-iam-policy-binding <target_acct> \
    --member='serviceAccount:<svc_acct>' \
    --role='roles/iam.serviceAccountTokenCreator'

# verify
gcloud iam service-accounts get-iam-policy <target_acct>
```

### Add SSH Key to Compute OS Login
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/os-login)
```
gcloud compute os-login ssh-keys add --key-file <public_key_file>
```

* * *

## Instance

### Instance - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/list)
```
gcloud compute instances list

# permissions required
compute.instances.list
compute.zones.list
```

### Instance - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/describe)
```
gcloud compute instances describe <instance_name> --zone <zone>

# permissions required
compute.instances.get
```

### Instance - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)
```
# example
gcloud compute instances create <instance_name> \
    --zone <zone> \
    --machine-type=e2-highmem-4 \
    --subnet=<subnet> \
    --tags=<tags_comma_separated> \
    --create-disk=auto-delete=yes,boot=yes,device-name=<instance_name>,image-family=debian-10,image-project=debian-cloud,mode=rw,size=60,type=pd-ssd

# write permissions required
compute.instances.create
compute.instances.setServiceAccount
compute.instances.setTags
compute.subnetworks.use
compute.subnetworks.useExternalIp
```

### Instance - Suspend
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/suspend)
```
gcloud compute instances suspend <instance_name> --zone <zone>

# permissions required
compute.instances.get
compute.instances.suspend
compute.zoneOperations.get
```

### Instance - Resume
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/resume)
```
gcloud compute instances resume <instance_name> --zone <zone>

# permissions required
compute.instances.get
compute.instances.resume
compute.zoneOperations.get
```

### Instance - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/delete)
```
gcloud compute instances delete <instance_name> --zone <zone>
  
# permissions required
compute.instances.delete
compute.zoneOperations.get
```

###  Instance - Delete Access Config
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/delete-access-config)
```
gcloud compute instances delete-access-config <instance_name> --zone <zone> \
    --access-config-name <access_config_name>

# permissions required
compute.instances.deleteAccessConfig
compute.instances.get
```

### Instance - Transfer Files
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/scp)
```
# to instance
gcloud compute scp <local_file> <instance_name>:<remote_dest> --zone <zone>

# from instance
gcloud compute scp <instance_name>:<remote_file> <local_dest> --zone <zone>

# permissions required
SSH access
compute.projects.get
compute.instances.get
```

### Instance Group - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instance-groups/unmanaged/create)
```
# unmanaged group
gcloud compute instance-groups unmanaged create <group_name> --zone <zone>

# write permissions required
compute.instanceGroups.create
```

### Instance Group - Add Instance
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instance-groups/unmanaged/add-instances)
```
# unmanaged group
gcloud compute instance-groups unmanaged add-instances <group_name> \
    --zone=<zone> \
    --instances=<instances_comma_separated>

# write permissions required
compute.instanceGroups.update
compute.instances.use
```

### Instance Group - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instance-groups/unmanaged/delete)
```
gcloud compute instance-groups unmanaged delete <group_name> --zone <zone>

# write permissions required
compute.instanceGroups.delete
```

* * *

## Disk / Snapshot

### Disk - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/disks/list)
```
gcloud compute disks list

# permissions required
compute.disks.list
```

### Disk - Attach to Instance
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/attach-disk)
```
gcloud compute instances attach-disk <instance_name> \
    --project <dst_proj> \
    --disk <disk_name> \
    --mode ro 

# permissions required
compute.disks.useReadOnly
compute.instances.attachDisk
compute.instances.get
compute.zoneOperations.get
iam.serviceAccounts.actAs

# example
gcloud compute instances attach-disk response \
    --project proj-298211 \
    --disk compromised-disk \
    --mode ro

# mount in destination instance
lsblk
sudo mkdir <mnt_pt>
sudo mount -o ro,noload /dev/<device_id> <mnt_pt>

# unmount in destination instance
sudo umount <mnt_pt>
```

### Snapshot - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/disks/snapshot)
```
gcloud compute disks snapshot <disk_name> \
    --project <src_proj> \
    --zone <src_zone> \ 
    --snapshot-names <ss_name>

# permissions required
compute.disks.createSnapshot
compute.snapshots.create
compute.snapshots.get
compute.zoneOperations.get

# example
gcloud compute disks snapshot compromised \
    --project compromised \
    --zone asia-southeast2-a \
    --snapshot-names compromised-ss
```

### Snapshot - Create Disk
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/disks/create)
```
gcloud compute disks create <disk_name> \ 
    --project <dst_proj> \
    --zone <dst_zone> \
    --source-snapshot <ss_path>

# permissions required
compute.disks.create
compute.disks.get
compute.snapshots.useReadOnly
compute.zoneOperations.get

# example
gcloud compute disks create compromised \
    --project response-298211 \
    --zone asia-southeast1-b \
    --source-snapshot projects/compromised/global/snapshots/compromised-ss
```

### Snapshot - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/snapshots/delete)
```
gcloud compute snapshots delete <ss_name> --project <src_proj>

# permissions required
compute.snapshots.delete
compute.globalOperations.get
```

* * *

## Network

### Network - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/list)
```
gcloud compute networks list

# permissions required
compute.networks.list
```

### Network - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/describe)
```
gcloud compute networks describe <network_name>

# permissions required
compute.networks.get
```

### Network - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/create)
```
gcloud compute networks create <network_name> --subnet-mode=custom

# write permissions required
compute.networks.create
```

### Network - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/delete)
```
gcloud compute networks delete <network_name>

# write permissions required
compute.networks.delete
```

### Subnet - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/create)
```
gcloud compute networks subnets create <subnet_name> \
    --network=<network_name> \
    --range=<subnet_range> \
    --region=<subnet_region>

# permissions required
compute.networks.get
compute.subnetworks.create
```

### Subnet - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/delete)
```
gcloud compute networks subnets delete <subnet_name> --region <subnet_region>

# write permissions required
compute.subnetworks.delete
```

### Subnet - Enable VPC Flow Logs
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/update)
```
gcloud compute networks subnets update <subnet_name> \ 
    --enable-flow-logs \
    --logging-aggregation-interval=interval-5-sec \
    --logging-flow-sampling=1.0 \
    --logging-metadata=include-all

# permissions required
compute.subnetworks.get
compute.subnetworks.update
```

### Subnet - Disable VPC Flow Logs
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/update)
```
gcloud compute networks subnets update <subnet_name> --no-enable-flow-logs

# permissions required
compute.subnetworks.get
compute.subnetworks.update
```

### Firewall Rule - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/list)
```
gcloud compute firewall-rules list

# permissions required
compute.firewalls.list
```

### Firewall Rule - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/describe)
```
gcloud compute firewall-rules describe <firewall_rule>

# permissions required
compute.firewalls.get
```

### Firewall Rule - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/create)
```
gcloud compute firewall-rules create <firewall_rule> \
    --network <network_name> \
    --action deny \
    --direction ingress \
    --rules tcp \
    --source-ranges 0.0.0.0/0 \
    --priority 1 \
    --target-tags <tags_comma_separated>
    
# permissions required
compute.firewalls.create
compute.firewalls.get
compute.networks.updatePolicy
```

### Firewall Rule - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/delete)
```
gcloud compute firewall-rules delete <firewall_rule>

# permissions required
compute.firewalls.delete
compute.networks.updatePolicy
compute.globalOperations.get
```

### Firewall Rule - Enable Logs
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/update)
```
gcloud compute firewall-rules update <firewall_rule> \
    --enable-logging \
    --logging-metadata=include-all

# permissions required
compute.firewalls.get
compute.firewalls.update
compute.networks.updatePolicy
```

### Firewall Rule - Disable Logs
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/update)
```
gcloud compute firewall-rules update <firewall_rule> --no-enable-logging

# permissions required
compute.firewalls.get
compute.firewalls.update
compute.networks.updatePolicy
```

### Network Tag - View Instances’
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/list)
```
gcloud compute instances list --format='table(name,status,tags.list())'
# OR
gcloud compute instances list --filter='tags:TAG_EXPRESSION'

# permissions required
compute.instances.list
compute.zones.list
```

### Network Tag - Add to Instance
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/add-tags)
```
gcloud compute instances add-tags <instance_name> --zone <zone> \
    --tags <tag_comma_separated>

# permissions required
compute.instances.get
compute.instances.setTags
```

### Network Tag - Remove from Instance
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/remove-tags)
```
gcloud compute instances remove-tags <instance_name> --zone <zone> \
    --tags <tag_comma_separated>

# permissions required
compute.instances.get
compute.instances.setTags
```

### VPC Network Peering - Create
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/peerings/create)
```
gcloud compute networks peerings create <peering_name> \
    --network=<network_name>
    --peer-project <dst_proj>
    --peer-network <dst_network_name>

# permissions required
compute.networks.addPeering
conmpute.networks.get
```

### VPC Network Peering - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/networks/peerings/delete)
```
gcloud compute networks peerings delete <peering_name> --network=<network_name>
    
# permissions required
compute.networks.removePeering
compute.networks.get
```

### Load Balancer - Create Regional Health Check
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/health-checks/create)
```
gcloud compute health-checks create http <healthcheck_name> \
    --region=<region> \
    --port=80
    
# write permissions required
compute.regionHealthChecks.create
```

### Load Balancer - Delete Regional Health Check
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/health-checks/delete)
```
gcloud compute health-checks delete <healthcheck_name> --region=<region>
    
# write permissions required
compute.regionHealthChecks.delete
```

### Load Balancer -  Create Backend Service
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/create)
```
gcloud compute backend-services create <backend_name> \
    --load-balancing-scheme=internal \
    --protocol=tcp \
    --region=<region> \
    --health-checks=<healthcheck_name> \
    --health-checks-region=<healthcheck_region>
    
# write permissions required
compute.regionBackendServices.create
```

### Load Balancer - Add Instance Group to Backend Service
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/add-backend)
```
gcloud compute backend-services add-backend <backend_name> \
    --region=<region> \
    --instance-group=<group_name> \
    --instance-group-zone=<group_zone>

# write permissions required
compute.regionBackendServices.update
```

### Load Balancer - Delete Backend Service
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/delete)
```
gcloud compute backend-services delete <backend_name> --region=<region>

# write permissions required
compute.regionBackendServices.delete
```

### Load Balancer - Create Forwarding Rule (Frontend Service)
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/forwarding-rules/create)
```
gcloud compute forwarding-rules create <rule_name> \
    --region=<region> \
    --load-balancing-scheme=internal \
    --backend-service=<backend_name> \
    --ports=all \
    --is-mirroring-collector \
    --network=<dst_network_name> \
    --subnet=<dst_subnet_name>
    
# write permissions required
compute.forwardingRules.create
compute.regionBackendServices.use
compute.subnetworks.use
```

### Load Balancer - Delete Forwarding Rule (Frontend Service)
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/forwarding-rules/delete)
```
gcloud compute forwarding-rules delete <rule_name> --region=<region>

# write permissions required
compute.forwardingRules.delete
```

### Packet Mirroring - Create Policy
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/packet-mirrorings/create)
```
gcloud compute packet-mirrorings create <policy_name> \
    --region=<region> \
    --network=projects/<src_project>/global/networks/<src_network> \
    --mirrored-tags=<tags_comma_separated> \
    --collector-ilb=<rule_name>

# write permissions required
compute.packetMirrorings.create
compute.networks.mirror (for dst proj only)
# + some unknown permission(s) in Compute Security Admin Role, likely one of the following
compute.firewallPolicies.copyRules
compute.firewallPolicies.move
compute.firewallPolicies.removeAssociation
compute.securityPolicies.addAssociation
compute.securityPolicies.copyRules
compute.securityPolicies.move
compute.securityPolicies.removeAssociation
# these permissions can't be added to a project Custom Role
# Use Compute Security Admin Role instead
```

### Packet Mirroring - Delete Policy
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/packet-mirrorings/delete)
```
gcloud compute packet-mirrorings delete <policy_name> --region=<region>

# write permissions required
compute.packetMirrorings.update
```

* * *

## Storage

### Bucket - List
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/ls)
```
gsutil ls

# permissions required
storage.buckets.list
```

### Bucket - Create
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/mb)
```
gsutil mb gs://<bucket_name> 

# permissions required
storage.buckets.create
```

### Bucket - List ACLs
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/acl)
```
gsutil acl get gs://<bucket_name> 

# permissions required
storage.buckets.get
storage.buckets.list
storage.buckets.getIamPolicy
```

### Bucket - Set ACLs
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/acl)  
[Predefined ACLs](https://cloud.google.com/storage/docs/access-control/lists#predefined-acl) - e.g. `private`, `publicRead`, `authenticatedRead`, etc
```
#  set bucket to private
gsutil acl set private gs://<bucket_name> 

# self-defined acls (textfile follows output of "gsutil acl get")
gsutil acl set <acl_textfile> gs://<bucket_name> 

# permissions required
storage.buckets.list
storage.buckets.setIamPolicy
storage.buckets.update
```

### Bucket - Grant Write Permission to Cloud-Storage-Analytics Group
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/iam)
```
gsutil iam ch group:cloud-storage-analytics@google.com:legacyBucketWriter gs://<bucket_name>

# permissions required
storage.buckets.get
storage.buckets.getIamPolicy
storage.buckets.setIamPolicy
```

### Bucket - Enable Logging for Target Bucket
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/logging)
```
gsutil logging set on -b gs://<log_storage_bucket> gs://<target_bucket>

# permissions required
storage.buckets.get
storage.buckets.update
```

### Bucket Object - List
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/ls)
```
gsutil ls gs://<bucket_name>

# permissions required
storage.objects.list
```

### Bucket Object - List ACLs
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/acl)
```
gsutil acl get gs://<object_name>

# permissions required
storage.buckets.get
storage.objects.get
storage.objects.getIamPolicy
storage.objects.list
```

### Bucket Object - Set ACLs
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/acl)  
[Predefined ACLs](https://cloud.google.com/storage/docs/access-control/lists#predefined-acl) - e.g. `private`, `publicRead`, `authenticatedRead`, etc
```
#  set bucket object to private
gsutil acl set private gs://<object_name>

# self-defined acls (textfile follows output of "gsutil acl get")
gsutil acl set <acl_textfile> gs://<object_name>

# permissions required
storage.buckets.list
storage.objects.list
storage.objects.setIamPolicy
storage.objects.update
```

### Bucket Object - List Metadata
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/stat)
```
gsutil stat gs://<object_name>

# permissions required    
storage.objects.get
```

### Bucket Object - Delete
[Reference](https://cloud.google.com/storage/docs/gsutil/commands/rm)
```
gsutil rm gs://<object_name>

# permissions required
storage.objects.delete
```

* * *

## IAM

### Custom Role - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/roles/list)
* Project must be specified to list only custom roles (even if gcp CLI default project had been configured), else all available/curated roles will be listed

```
gcloud iam roles list \ 
    --project <project> \
    --format="table(title, name, description, stage, etag)"

# permissions required
iam.roles.list
```

### Custom Role - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/roles/describe)
* Project must be specified (even if gcp CLI default project had been configured)
* Specify role name not role title

```
gcloud iam roles describe <role_name> --project <project>

# permissions required
iam.roles.get
```

### Custom Role - Disable
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/roles/update)
* Project must be specified (even if gcp CLI default project had been configured)
* Specify role name not role title

```
gcloud iam roles update <role_name> --project <project> --stage=DISABLED

# permissions required
iam.roles.update
iam.roles.get
```

### Custom Role - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/roles/delete)
* Project must be specified (even if gcp CLI default project had been configured)
* Specify role name not role title

```
gcloud iam roles delete <role_name> --project <project>

# permissions required
iam.roles.delete
```

### Service Account - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/list)
* Only service accounts created in the project
* Excludes service accounts from other projects added to this project

```
gcloud iam service-accounts list

# permissions required
iam.serviceAccounts.list
```

### Service Account - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/describe)
```
gcloud iam service-accounts describe <svc_acct>

# permissions required
iam.serviceAccounts.get
```

### Service Account - List Roles
[Reference](https://cloud.google.com/sdk/gcloud/reference/projects/get-iam-policy)
```
gcloud projects get-iam-policy <project> \
    --flatten="bindings[].members" \
    --format='table(bindings.role)' \
    --filter="bindings.members:<svc_acct>"

# permissions required
resourcemanager.projects.getIamPolicy
```

### Service Account - Describe IAM Policy
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/get-iam-policy)
```
gcloud iam service-accounts get-iam-policy <svc_acct>

# permissions required
iam.serviceAccounts.getIamPolicy
```

### Service Account - Remove  IAM Policy Binding
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/remove-iam-policy-binding)
```
gcloud iam service-accounts remove-iam-policy-binding <svc_acct> \
    --member='<serviceAccount_or_user>:Maccount>' \
    --role='roles/<role>'

# permissions required
iam.serviceAccounts.getIamPolicy
iam.serviceAccounts.setIamPolicy


# example
# remove iam.serviceAccountTokenCreator from service account
gcloud iam service-accounts remove-iam-policy-binding totest@compromised.iam.gserviceaccount.com \
    --member='serviceAccount:svc_acct@response-298211.iam.gserviceaccount.com' \
    --role='roles/iam.serviceAccountTokenCreator'

# remove iam.serviceAccountUser from user account
gcloud iam service-accounts remove-iam-policy-binding totest@compromised.iam.gserviceaccount.com \
    --member='user:user@gmail.com' \
    --role='roles/iam.serviceAccountUser'
```

### Service Account - Disable
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/disable)
```
gcloud iam service-accounts disable <svc_acct>

# permissions required
iam.serviceAccounts.disable
```

### Service Account - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/delete)
```
gcloud iam service-accounts delete <svc_acct>

# permissions required
iam.serviceAccounts.delete
```

### Service Account Key - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/keys/list)
```
gcloud iam service-accounts keys list --iam-account <svc_acct>

# permissions required
iam.serviceAccountKeys.list
```

### Service Account Key - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/keys/delete)
```
gcloud iam service-accounts keys delete <key_id> --iam-account <svc_acct>

# permissions required
iam.serviceAccountKeys.delete
```

### User Account - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/projects/get-iam-policy)
```
gcloud projects get-iam-policy <project> | grep -i user: | sort | uniq

# permissions required
resourcemanager.projects.getIamPolicy
```

### User Account - List Roles
[Reference](https://cloud.google.com/sdk/gcloud/reference/projects/get-iam-policy)
```
gcloud projects get-iam-policy <project> \
    --flatten="bindings[].members" \
    --format='table(bindings.role)' \
    --filter="bindings.members:<user_acct>"

# permissions required
resourcemanager.projects.getIamPolicy
```

### User Account - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/projects/remove-iam-policy-binding)
```
gcloud projects remove-iam-policy-binding <project> \
    --member=user:<user_acct> \
    --role=roles/<role>

# alternative
# save project IAM policy to yaml file
gcloud projects get-iam-policy <project> --format yaml > users.yaml

# remove the user from his role in the yaml file, e.g. (see user account enclosed by double tildes ~~)
bindings:
- members:
  - serviceAccount:service-2160192374724@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:2160192374724-compute@developer.gserviceaccount.com
  - serviceAccount:2160192374724@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:user1@gmail.com
  ~~- user:user2@gmail.com~~
  role: roles/owner
etag: BwW2uSt0iE0=
version: 1

# set new project IAM policy with the modified yaml file
gcloud projects set-iam-policy <project> users.yaml

# permissions required
resourcemanager.projects.getIamPolicy
resourcemanager.projects.setIamPolicy
```

### Project-wide SSH Key - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/project-info/describe)
```
gcloud compute project-info describe

# permissions required
compute.projects.get
```

### Project-wide SSH Key - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/project-info/add-metadata)  
[Reference to steps](https://cloud.google.com/compute/docs/connect/restrict-ssh-keys#gcloud_1)
```
gcloud compute project-info add-metadata --metadata-from-file=ssh-keys=<file>

# permissions required
compute.globalOperations.get
compute.projects.get
compute.projects.setCommonInstanceMetadata
iam.serviceAccounts.actAs
```

### Instance-specific SSH Keys - Describe
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/describe)
```
gcloud compute instances describe <instance_name> --zone <zone> \
    --format="value(metadata)"

# permissions required
compute.instances.get
```

### Instance-specific SSH Key - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/compute/instances/add-metadata)  
[Reference to steps](https://cloud.google.com/compute/docs/connect/restrict-ssh-keys#gcloud_2)
```
gcloud compute instances add-metadata <instance_name> --metadata-from-file ssh-keys=<file>

# permissions required
compute.instances.get
compute.instances.setMetadata
compute.zoneOperations.get
iam.serviceAccounts.actAs
```

* * *

## Logging

### Logs - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/logging/logs/list)
```
gcloud logging logs list

# permissions required
logging.logs.list
```

### Logs - Query
[Reference](https://cloud.google.com/sdk/gcloud/reference/logging/read)  
[Reference to building queries](https://cloud.google.com/logging/docs/view/building-queries)
```
# manual
gcloud logging read <log_filter>

# permissions required
Logs Viewer role
Private Logs Viewer role

# example
gcloud logging read "resource.type=gce_instance AND textPayload:SyncAddress" --limit 10 --format json
```

### Log Sink - List
[Reference](https://cloud.google.com/sdk/gcloud/reference/logging/sinks/list)
```
gcloud logging sinks list

# permissions required
logging.sinks.list
```

### Log Sink - Update
[Reference](https://cloud.google.com/sdk/gcloud/reference/logging/sinks/update)
```
gcloud logging sinks update <sink_name> --log-filter='<filter>'

# permissions required
logging.sinks.get
logging.sinks.update
```

### Log Sink - Delete
[Reference](https://cloud.google.com/sdk/gcloud/reference/logging/sinks/delete)
```
gcloud logging sinks delete <sink>

# permissions required
logging.sinks.delete
```

##  Kubernetes

### Cluster - Connect
[Reference](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
```
gcloud container clusters get-credentials <cluster_name> --zone <zone>

# permissions required
container.clusters.get
```

### Pod - Exec
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec)
```
kubectl exec pod <pod_name> -- <command>

# permissions required
container.pods.exec
container.pods.get
```

### Pod - Label
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label)
```
kubectl label pods <pod_name> <label>=<true/false>

# permissions required
container.pods.get
container.pods.update

# example
kubectl label pods nginx-deployment-c9445c769-ll8nv quarantine=true
```

### Node - Cordon
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cordon)
```
kubectl cordon <node_name>

# permissions required
container.nodes.get
container.nodes.update
```

### Node - Drain
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#drain)
```
kubectl drain <node_name> --pod-selector=<selector>

# permissions required
container.daemonSets.get
container.nodes.get
container.pods.evict
container.pods.list

# example
kubectl drain gke-ir-test-cluster-default-pool-0f6491d0-x8vv --pod-selector='!quarantine'
```

### Service - List
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get)
```
kubectl get services

# permissions required
container.services.list
```

### Service - Delete
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete)
```
kubectl delete service <service_name>

# permissions required
container.services.delete
```

### Service - Patch
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#patch)
```
kubectl patch service <service-name> -p '{"spec":{"selector":{"<key>": "<value>"}}}'

# permissions required
container.services.get
container.services.update
```

### Network Policy - List
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get)
```
kubectl get networkpolicies

# permissions required
container.networkPolicies.list
```

### Network Policy - Create
[Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)
```
kubectl apply -f <networkpolicy_yaml>

# permissions required
container.networkPolicies.create
container.networkPolicies.get
```