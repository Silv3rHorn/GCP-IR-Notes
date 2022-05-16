# Minimal Role / Permissions for Target

```{admonition} Assumption
*Service account had been created and granted the necessary Viewer roles (`Organization Viewer`, `Folder Viewer`, `Viewer`, `Private Logs Viewer`)*
```

An Incident Response team is likely to encounter resistance from engineering / business teams  when requesting **non-read** permissions to target (potential compromised) GCP Organisation and Projects due to potential production impact.

Hence, to reduce such concerns while still enabling effective incident response, a set of minimal **non-read** permissions is provided here.

```{warning}
Provided minimal permissions are not meant to be a complete set of permissions necessary to perform any incident response investigation on GCP, for e.g. Cloud SQL investigations were not catered for. However, they should be sufficient for > 90% of incidents.
```

## Grant Permissions Guide

The Organisation / Project owner is expected to use [gcloud cli](https://cloud.google.com/sdk/gcloud) and [this yml file](https://raw.githubusercontent.com/Silv3rHorn/GCP-IR-Notes/master/admin/svc_acct/min_perm_list_target.yml) that contains the minimum permissions to perform the following steps.

To grant the permissions **org-wide**:
```shell
# create a custom role with the required permissions
# when prompted to confirm the creation, input Y (Yes)
gcloud iam roles create <role_id> --organization=<org_id> --file=min_perm_list_target.yml

# grant roles to the service account
# when prompted to specify a condition, select None
gcloud organizations add-iam-policy-binding <organization_id> \
    --member=serviceAccount:<svc_acct> \
    --role=organizations/<org_id>/roles/<role_id>
```
To grant permissions for a **specific project**:
```shell
# create a custom role with the required permissions
# when prompted to confirm the creation, input Y (Yes)
gcloud iam roles create <role_id> --project=<project_id> --file=min_perm_list_target.yml

# grant roles to the service account
# when prompted to specify a condition, select None
gcloud projects add-iam-policy-binding <project_id> \
    --member=serviceAccount:<svc_acct> \
    --role=projects/<project_id>/roles/<role_id>
```

## Permissions Usage
````{div} full-width
```{list-table}
:header-rows: 1

*   - Permissions
    - Used in
    - Required for
*   - ***compute.disks***
    - 
    - 
*   - `compute.disks.create`
    - `gcloud compute disks create`
    - Create disk from snapshot
*   - `compute.disks.createSnapshot`
    - `gcloud compute disks snapshot`
    - Create snapshot from disk
*   - ***compute.firewalls***
    - 
    - 
*   - `compute.firewalls.create`
    - `gcloud compute firewall-rules create`
    - Create firewall rules
*   - `compute.firewalls.delete`
    - `gcloud compute firewall-rules delete`
    - Delete firewall rules
*   - `compute.firewalls.update`
    - `gcloud compute firewall-rules update`
    - Enable firewall rule logs
*   - ***compute.instances***
    - 
    - 
*   - `compute.instances.delete`
    - `gcloud compute instances delete`
    - Delete compute instances
*   - `compute.instances.deleteAccessConfig`
    - `gcloud compute instances delete-access-config`
    - Remove external IP address of compute instance
*   - `compute.instances.resume`
    - `gcloud compute instances resume`
    - Resume compute instances
*   - `compute.instances.setMetadata`
    - `gcloud compute instances add-metadata`
    - Delete instance-specific SSH keys
*   - `compute.instances.setTags`
    - `gcloud compute instances add-tags`  
      `gcloud compute instances remove-tags`
    - Add network tag to compute instance  
      Remove network tag from compute instance
*   - `compute.instances.suspend`
    - `gcloud compute instances suspend`
    - Suspend compute instances
*   - ***compute.networks***
    - 
    - 
*   - `compute.networks.addPeering`
    - `gcloud compute network peerings create`
    - Create VPC network peering for packet capture
*   - `compute.networks.updatePolicy`
    - `gcloud compute firewall-rules create`  
      `gcloud compute firewall-rules delete`  
      `gcloud compute firewall-rules update`
    - Create, update and delete firewall rules
*   - ***compute.subnetworks***
    - 
    - 
*   - `compute.subnetworks.update`
    - `gcloud compute networks subnets update`
    - Enable/Disable VPC flow logs
*   - ***compute.projects***
    - 
    - 
*   - `compute.projects.setCommonInstanceMetadata`
    - `gcloud compute project-info add-metadata`
    - Delete project-wide SSH key
*   - ***compute.snapshots***
    - 
    - 
*   - `compute.snapshots.create`
    - `gcloud compute disks snapshot`
    - Create snapshot from disk
*   - `compute.snapshots.delete`
    - `gcloud compute snapshots delete`
    - Delete snapshot
*   - ***iam.roles***
    - 
    - 
*   - `iam.roles.delete`
    - `gcloud iam roles delete`
    - Delete IAM custom role
*   - `iam.roles.update`
    - `gcloud iam roles update`
    - Disable IAM custom role
*   - ***iam.serviceAccounts***
    - 
    - 
*   - `iam.serviceAccounts.actAs`
    - `gcloud compute project-info add-metadata`  
      `gcloud compute instances add-metadata`
    - Delete project-wide/instance-specific SSH key
*   - `iam.serviceAccounts.delete`
    - `gcloud iam service-accounts delete`
    - Delete IAM service account
*   - `iam.serviceAccounts.disable`
    - `gcloud iam service-accounts disable`
    - Disable IAM service account
*   - `iam.serviceAccounts.setIamPolicy`
    - `gcloud iam service-accounts remove-iam-policy-binding`
    - Remove service account IAM policy binding
*   - ***iam.serviceAccountKeys***
    - 
    - 
*   - `iam.serviceAccountKeys.delete`
    - `gcloud iam service-accounts keys delete`
    - Delete IAM service account key
*   - ***logging.sinks***
    -
    -
*   - `logging.sinks.delete`
    - `gcloud logging sinks delete`
    - Delete logging sinks
*   - `logging.sinks.update`
    - `gcloud logging sinks update`
    - Update logging sinks
*   - ***resourcemanager.projects***
    - 
    - 
*   - `resourcemanager.projects.setIamPolicy`
    - `gcloud projects remove-iam-policy-binding`
    - Delete user account from project
*   - ***storage.buckets***
    - 
    - 
*   - `storage.buckets.create`
    - `gsutil mb`
    - Create bucket for log storage
*   - `storage.buckets.get`
    - `gsutil acl get`  
      `gsutil iam ch`
      <br></br>
      `gsutil logging set on`
    - List bucket (object) ACLs  
      Grant bucket write permission to Cloud-Storage-Analytics  
      Enable logging for target bucket
*   - `storage.buckets.getIamPolicy`
    - `gsutil acl get`  
      `gsutil iam ch`
    - List bucket ACLs  
      Grant bucket write permission to Cloud-Storage-Analytics
*   - `storage.buckets.setIamPolicy`
    - `gsutil acl set`  
      `gsutil iam ch`
    - Set bucket ACLs  
      Grant bucket write permission to Cloud-Storage-Analytics
*   - `storage.buckets.update`
    - `gsutil acl set`  
      `gsutil logging set on`
    - Set bucket ACLs  
      Enable logging for target bucket
*   - ***storage.objects***
    - 
    - 
*   - `storage.objects.delete`
    - `gsutil delete`
    - Delete bucket object
*   - `storage.objects.get`
    - `gsutil acl get`  
      `gsutil stat`
    - List bucket object ACLs  
      List bucket object metadata
*   - `storage.objects.getIamPolicy`
    - `gsutil acl get`
    - List bucket object ACLs
*   - `storage.objects.list`
    - `gsutil ls`  
      `gsutil acl get`  
      `gsutil acl set`
    - List bucket objects  
      List bucket object ACLs  
      Set bucket object ACLs
*   - `storage.objects.setIamPolicy`
    - `gsutil acl set`
    - Set bucket object ACLs
*   - `storage.objects.update`
    - `gsutil acl set`
    - Set bucket object ACLs
*   - ***container***
    - 
    - 
*   - `container.networkPolicies.create`
    - `kubectl apply -f <network-policy>`
    - Apply k8s network policy
*   - `container.nodes.update`
    - `kubectl cordon`
    - Cordon compromised nodes
*   - `container.pods.evict`
    - `kubectl drain`
    - Drain pods from nodes
*   - `container.pods.exec`
    - `kubectl exec pod`
    - Execute commands from pods
*   - `container.pods.update`
    - `kubectl label pods`
    - Label compromised pods for isolation
*   - `container.services.delete`
    - `kubectl delete service`
    - Delete k8s load balancer service
*   - `container.services.update`
    - `kubectl patch service`
    - Update k8s service
```
````