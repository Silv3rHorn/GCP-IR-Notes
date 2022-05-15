# Minimal Role / Permissions for Analyst
```{admonition} Assumption
*Service account had been created and granted the necessary Viewer roles (`Organization Viewer`, `Folder Viewer`, `Viewer`, `Private Logs Viewer`)*
```

Besides the [non-read permissions at the **target organisation / project**](min_perm_target.md), the service account also requires a specific set of **non-read** permissions at the **analyst project** to perform operations (e.g. create firewall rules, attach disks) to facilitate incident response.

## Grant Permissions Guide

To simplify matters, the service account can be granted `Editor` role in the analyst project. If that is not possible, a specific set of minimal permissions and `Compute Security Admin` role can be granted using [gcloud cli](https://cloud.google.com/sdk/gcloud) and [this yml file](./min_perm_list_analyst) to the service account.

```shell
# create a custom role with the required permissions
# when prompted to confirm the creation, input Y (Yes)
gcloud iam roles create <role_id> --project=<project_id> --file=min_perm_list_analyst.yml

# grant created role to the service account
# when prompted to specify a condition, select None
gcloud projects add-iam-policy-binding <project_id> \
    --member=serviceAccount:<svc_acct> \
    --role=projects/<project_id>/roles/<role_id>

# grant Compute Security Admin role to the service account
# when prompted to specify a condition, select None
gcloud projects add-iam-policy-binding <project_id> \
    --member=serviceAccount:<svc_acct> \
    --role=roles/compute.securityAdmin
```
## Permissions Usage
````{div} full-width
```{list-table}
:header-rows: 1

*   - Permissions
    - Used in
    - Required for
*   - ***compute.firewalls***
    - 
    - 
*   - `compute.firewalls.create`
    - `gcloud compute firewall-rules create`
    - Packet Mirroring (firewall rule)
*   - ***compute.forwardingRules***
    - 
    - 
*   - `compute.forwardingRules.create`
    - `gcloud compute forwarding-rules create`
    - Packet Mirroring (load balancer)
*   - `compute.forwardingRules.delete`
    - `gcloud compute forwarding-rules delete`
    - Packet Mirroring (load balancer)
*   - ***compute.instanceGroups***
    - 
    - 
*   - `compute.instanceGroups.create`
    - `gcloud compute instance-groups unmanaged create`
    - Packet Mirroring (load balancer)
*   - `compute.instanceGroups.delete`
    - `gcloud compute instance-groups unmanaged delete`
    - Packet Mirroring (load balancer)
*   - `compute.instanceGroups.update`
    - `gcloud compute instance-groups unmanaged add-instances`
    - Packet Mirroring (load balancer)
*   - ***compute.instances***
    - 
    - 
*   - `compute.instances.attachDisk`
    - `gcloud compute instances attach-disk`
    - Attach disk to compute instance
*   - `compute.instances.create`
    - `gcloud compute instances create`
    - Packet Mirroring (load balancer)
*   - `compute.instances.setServiceAccount`
    - `gcloud compute instances create`
    - Packet Mirroring (load balancer)
*   - `compute.instances.setTags`
    - `gcloud compute instances create`
    - Packet Mirroring (load balancer)
*   - `compute.instances.use`
    - `gcloud compute instance-groups unmanaged add-instances`
    - Packet Mirroring (load balancer)
*   - ***compute.networks***
    - 
    - 
*   - `compute.networks.addPeering`
    - `gcloud compute network peerings create`
    - Packet Mirroring (VPC network peering)
*   - `compute.networks.create`
    - `gcloud compute networks create`
    - Packet Mirroring (VPC network peering)
*   - `compute.networks.delete`
    - `gcloud compute networks delete`
    - Packet Mirroring (VPC network peering)
*   - ***compute.packetMirrorings***
    - 
    - 
*   - `compute.packetMirrorings.update`
    - `gcloud compute packet-mirrorings delete`
    - Packet Mirroring (Policy)
*   - ***compute.regionBackendServices***
    - 
    - 
*   - `compute.regionBackendServices.create`
    - `gcloud compute backend-services create`
    - Packet Mirroring (load balancer)
*   - `compute.regionBackendServices.delete`
    - `gcloud compute backend-services delete`
    - Packet Mirroring (load balancer)
*   - `compute.regionBackendServices.update`
    - `gcloud compute backend-services add-backend`
    - Packet Mirroring (load balancer)
*   - `compute.regionBackendServices.use`
    - `gcloud compute forwarding-rules create`
    - Packet Mirroring (load balancer)
*   - ***compute.regionHealthChecks***
    - 
    - 
*   - `compute.regionHealthChecks.create`
    - `gcloud compute health-checks create`
    - Packet Mirroring (load balancer)
*   - `compute.regionHealthChecks.delete`
    - `gcloud compute health-checks delete`
    - Packet Mirroring (load balancer)
*   - ***compute.subnetworks***
    - 
    - 
*   - `compute.subnetworks.create`
    - `gcloud compute networks subnets create`
    - Packet Mirroring (VPC network peering)
*   - `compute.subnetworks.delete`
    - `gcloud compute networks subnets delete`
    - Packet Mirroring (VPC network peering)
*   - `compute.subnetworks.use`
    - `gcloud compute forwarding-rules create`  
      `gcloud compute instances create`
    - Packet Mirroring (load balancer)
*   - `compute.subnetworks.useExternalIp`
    - `gcloud compute instances create`
    - Packet Mirroring (load balancer)
```
````