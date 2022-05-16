# Detections

## Creation of GPU-Equipped VMs
- Often performed by adversaries whose objective is to perform cryptomining
```
resource.type="gce_instance"
protoPayload.methodName=beta.compute.instances.insert
protoPayload.request.guestAccelerators.acceleratorType:*

```

## Reduction in Retention of Log Sink Bucket
- Applies to log buckets whose retention had been reduced to < 30 days
- Not applicable to `_Required` log bucket since retention can't be modified
```
resource.type="audited_resource"
protoPayload.methodName=google.logging.v2.ConfigServiceV2.UpdateBucket
protoPayload.request.updateMask="retentionDays"
protoPayload.request.bucket.retentionDays<30
```

## Generation of Access Token
- By default disabled. Enable `Admin Read` for Identity and Access Management (IAM) API
```
resource.type="service_account" protoPayload.methodName="GenerateAccessToken"
```

## Usage of Generated Access Token or Account Impersonation
```
protoPayload.authenticationInfo.serviceAccountDelegationInfo:*
```
