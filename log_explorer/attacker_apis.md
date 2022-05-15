# Attackers' APIs

* This is an attempt to list corresponding GCP method names of APIs specified in [AWS mindmap by Expel](https://twitter.com/jhencinski/status/1328714145848549376/photo/1)
* How to use for searches in GCP Log Explorer
    * `resource.type="<value in GCP Equiv resource.type>"`
    * If `GCP Equiv methodName` contains a version number of the method (e.g. `v1`)
        * `protoPayload.methodName:"<value after last . (period) in GCP Equiv methodName>"` (e.g. `protoPayload.methodName:"CreateFunction"` for `google.cloud.functions.v1.CloudFunctionsService.CreateFunction`)
    * If `GCP Equiv methodName` is prefixed with `beta`
        * `protoPayload.methodName:"<value in GCP Equiv methodName excluding beta.>"`
        * Possibility that `methodName` might change in the future once out of beta
    * If `GCP Equiv methodName` is not any of the above
        * `protoPayload.methodName:"<value in GCP Equiv methodName>"`
<style>
td {
  font-size: 10px
}
</style>
## Initial Access
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - Console/CLI
    - Console/CLI
    - ConsoleLogin
    - NA
    - NA
    - Use Google Workspace logs
*   - 
    - 
    - GetFederationToken
    - NA
    - NA
    - NIL
*   - 
    - 
    - GetSessionToken
    - service_account
    - GenerateAccessToken
    - Require `Data Read` & `Data Write` to be enabled for Identity and Access Management (IAM) API
*   - 
    - 
    - StartSession
    - NA
    - NA
    - NIL
*   - 
    - 
    - GetAuthorizationToken
    - NA
    - NA
    - NIL
```
````

## Credential Access
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - Secrets Manager
    - Secret Manager
    - GetSecretValue
    - audited_resource
    - google.cloud.secretmanager.v1.SecretManagerService.GetSecret
    - NIL
*   - EC2
    - GCE
    - GetPasswordData
    - NA
    - NA
    - NIL
```
````

## Privilege Escalation
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - Lambda
    - GCF
    - CreateFunction
    - cloud_function
    - google.cloud.functions.v1.CloudFunctionsService.CreateFunction
    - NIL
*   - 
    - 
    - UpdateFunctionConfiguration
    - cloud_function
    - google.cloud.functions.v1.CloudFunctionsService.UpdateFunction
    - NIL
*   - 
    - 
    - UpdateFunctionCode
    - cloud_function
    - same as above
    - NIL
*   - IAM
    - IAM
    - CreateUser
    - project
    - SetIamPolicy
    - NIL
*   - 
    - 
    - CreateLoginProfile
    - NA
    - NA
    - NIL
*   - 
    - 
    - UpdateProfile
    - NA
    - NA
    - NIL
*   - 
    - 
    - PutUserPolicy
    - Depends on resource type (for e.g.):  
    **GCE Instance** - gce_instance  
    **GCS Bucket** - gcs_bucket  
    **Cloud Function** - cloud_function
    - Depends on resource type (for e.g.):  
    **GCE Instance** - v1.compute.instances.setIamPolicy  
    **GCS Bucket** - storage.setIamPermissions  
    **Cloud Function** - google.cloud.functions.v1.CloudFunctionsService.SetIamPolicy
    - Policy is applied to resources in GCP
*   - 
    - 
    - AttachUserPolicy
    - same as above
    - same as above
    - same as above
*   - 
    - 
    - AddUserToGroup
    - NA
    - NA
    - Use Google Workspace logs
*   - 
    - 
    - PutGroupPolicy
    - NA
    - NA
    - same as above
*   - 
    - 
    - CreateGroup
    - NA
    - NA
    - same as above
*   - 
    - 
    - CreateRole
    - iam_role
    - google.iam.admin.v1.CreateRole
    - NIL
*   - 
    - 
    - AssumeRole
    - service_account
    - GenerateAccessToken
    - Require `Data Read` & `Data Write` to be enabled for Identity and Access Management (IAM) API
*   - 
    - 
    - UpdateAssumeRolePolicy
    - iam_role
    - google.iam.admin.v1.UpdateRole
    - NIL
*   - 
    - 
    - CreateInstanceProfile
    - NA
    - NA
    - Any available service account can be assigned to a VM instance in GCP
*   - 
    - 
    - AttachRolePolicy
    - NA
    - NA
    - Policies are only attached to resources in GCP
*   - 
    - 
    - PutRolePolicy
    - NA
    - NA
    - same as above
*   - 
    - 
    - CreatePolicy
    - NA
    - NA
    - No API to create IAM managed policy in GCP
*   - 
    - 
    - CreatePolicyVersion
    - NA
    - NA
    - same as above
```
````

## Discovery
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - All
    - All
    - Head*
    - All
    - NA
    - NIL
*   - 
    - 
    - Get*
    - All
    - NA
    - NIL
*   - 
    - 
    - List*
    - All
    - *.list  
    *.aggregatedList
    - Might require `Data Read` API to be enabled
*   - 
    - 
    - Describe*
    - All
    - *.get
    - NIL
```
````

## Persistence
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - Lambda
    - GCF
    - CreateFunction
    - cloud_function
    - google.cloud.functions.v1.CloudFunctionsService.CreateFunction
    - NIL
*   - 
    - 
    - UpdateFunctionCode
    - cloud_function
    - google.cloud.functions.v1.CloudFunctionsService.UpdateFunction
    - NIL
*   - IAM
    - IAM
    - CreateUser
    - project
    - SetIamPolicy
    - NIL
*   - 
    - 
    - CreateRole
    - iam_role
    - google.iam.admin.v1.CreateRole
    - NIL
*   - 
    - 
    - UpdateAssumeRolePolicy
    - iam_role
    - google.iam.admin.v1.UpdateRole
    - NIL
*   - 
    - 
    - CreateAccessKey
    - service_account
    - google.iam.admin.v1.CreateServiceAccountKey
    - Access keys can only be created for service accounts in GCP
*   - EC2
    - GCE
    - CreateInstance
    - gce_instance
    - beta.compute.instances.insert
    - NIL
*   - 
    - 
    - CreateKeyPair
    - NA
    - NA
    - NIL
*   - 
    - 
    - CreateImage
    - api
    - beta.compute.machineImages.insert
    - NIL
*   - ECR
    - Container Registry
    - CreateRepository
    - NIL
    - NIL
    - Audit logging does not seem to be available; only storage bucket creation event is logged
*   - 
    - 
    - PutImage
    - NIL
    - NIL
    - see above
*   - 
    - Artifact Registry
    - CreateRepository
    - audited_resource
    - google.devtools.artifactregistry.v1.ArtifactRegistry.CreateRepository
    - NIL
*   - 
    - 
    - PutImage
    - audited_resource
    - Docker-PutManifest
    - Require `Data Read` to be enabled for Artifact Registry API
```
````

## Collection
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - S3
    - Cloud Storage
    - CopyObject
    - gcs_bucket
    - storage.objects.create
    - Require `Data Read` to be enabled for Google Cloud Storage API 
*   - 
    - 
    - GetObject
    - gcs_bucket
    - storage.objects.get
    - Require `Data Read` & `Data Write` to be enabled for Google Cloud Storage API
*   - EC2
    - GCE
    - GetConsoleScreenshot
    - gce_instance
    - v1.compute.instances.getScreenshot
    - NIL
*   - 
    - 
    - CreateSnapshot
    - gce_disk
    - v1.compute.disks.createSnapshot
    - NIL
```
````

## Exfiltration
````{div} full-width
```{list-table}
:header-rows: 1

*   - <font size="2">AWS Service</font>
    - <font size="2">GCP Equiv</font>
    - <font size="2">AWS API</font>
    - <font size="2">GCP Equiv `resource.type`</font>
    - <font size="2">GCP Equiv `methodName`</font>
    - <font size="2">Remarks</font>
*   - AMI
    - Machine Images
    - ModifySnapshotAttribute
    - NA
    - NA
    - Snapshots (if exists) does not seem accessible from Machine Image in GCP
*   - 
    - 
    - ModifyImageAttribute
    - NA
    - NA
    - Machine images does not seem to be modifiable in GCP
*   - EBS
    - Block Storage
    - ModifySnapshotAttribute
    - gce_snapshot
    - beta.compute.snapshots.setIamPolicy
    - NIL
*   - S3
    - Cloud Storage
    - PutBucketPolicy
    - gcs_bucket
    - storage.setIamPermissions
    - NIL
*   - 
    - 
    - PutBucketAcl
    - gcs_bucket
    - storage.setIamPermissions
    - NIL
*   - RDS
    - Cloud SQL
    - RestoreDBInstanceFromDBSnapshot
    - cloudsql_database
    - cloudsql.instances.restoreBackup
    - NIL
*   - 
    - Cloud Spanner
    - RestoreDBInstanceFromDBSnapshot
    - spanner_instance
    - google.spanner.admin.database.v1.DatabaseAdmin.RestoreDatabase
    - NIL
```
````