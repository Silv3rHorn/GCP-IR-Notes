# Playbook

````{div} full-width
```{note}
Kubernetes Dashboard and API server (ports 10250/10255) exposures, and control-plane attacks are unlikely to happen due to GKE's default security configuration. Hence, they are given little or no focus in this playbook.
```
```{image} playbook_1.png
:name: playbook_1
```
**References**  
* [Threat matrix for Kubernetes](https://www.microsoft.com/security/blog/2020/04/02/attack-matrix-kubernetes/)
* [Secure containerized environments with updated threat matrix for Kubernetes](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)

````

## Breakdown

````{div} full-width
```{list-table}
:header-rows: 1

*   - Phase
    - Activity
    - Description
*   - Detection
    - Incident escalated involving GKE
    - **Possible escalation sources:**  
    **SOC** - Detections from use cases  
    **CTI** - Threat intel reports  
    **End user** - Unusual activity (e.g. higher billing cost, unfamiliar deployments) noticed  
    **3rd party** - Found public exposure of sensitive information or unpatched vulnerability
    <br/><br/>
    If root cause is not known from escalation source, proceed in parallel to `Determine root cause`.
*   - Analysis
    - Identify GCP project, cluster and owner/PIC
    - **Identify GCP Project & Cluster**  
    One way is to use the `Search products and resources` search in GCP menu bar to search for IP Addresses and Names of resources. Note that the search is limited to only the resources you have access to.
    <br/><br/>
    **Identify Owner/PIC**  
    One way is to access https://console.cloud.google.com/iam-admin/iam?project=<project_id> and user account with the role `Owner`
    <br/><br/>
    Request Owner/PIC to grant the IR team the [necessary permissions](../admin/svc_acct/min_perm_target.md) at his/her project and SSH access to the nodes in the GKE cluster under investigation
*   - 
    - Request for preservation of evidence
    - Especially important given the self-healing and auto-scaling capabilities of Kubernetes which contributes to the volatility
    <br/><br/>
    Request Owner/PIC **NOT** to <ul><li>Delete/restart the impacted cluster/deployment/node/pod(s)</li><li>Perform any remediation activities unless it had been cleared by the IR team</li></ul>
*   - 
    - Discover cluster
    - Refer to [Discover Cluster](./playbook/discover_cluster.ipynb)
*   - 
    - Artifacts [logs] in persistent storage?
    - This impacts how the artifacts would be collected for analysis
    <br/><br/>
    **Potential persistent storage locations:**  
    `gcePersistentDisk` - Google Compute Engine Volume  
    `PersistentVolumeClaim` - Makes a claim from the cluster for an allocation of persistent storage (provided by a matching `PersistentVolume` object)  
    `hostPath` - Uses a file or directory on the node to emulate network-attached storage
    <br/><br/>
    Refer to [Application Logs](./playbook/app_logs.md)
*   - 
    - Acquire persistent storage
    - Refer to [Application Logs](./playbook/app_logs.md)
*   - 
    - Acquire **node** persistent storage
    - Refer to [Application Logs](./playbook/app_logs.md)
*   - 
    - Perform live response
    - Refer to [Live Response](./playbook/analysis/live_response.md)
*   - 
    - Determine root cause
    - **Compromised Credentials**<ul><li>Compromised Google Account</li><li>Access key (e.g. service account key or SSH key) exposed to public</li><li>Adversary obtaining valid kubeconfig file (e.g. via a compromised endpoint) which they can use for accessing the clusters</li></ul>
    **Compromised Image**<ul><li>Adversary plants their own compromised image(s) in a private registry (e.g. within the GCP project) they obtained access to. These images can then be pulled by a user</li><li>Using untrusted images from public registries (e.g. Docker Hub) that may be malicious</li></ul>
    **Unintended Exposure**<ul><li>Interfaces that are not intended to be exposed to the Internet, and therefore don’t require (secure) authentication by default. Exposing them to the Internet allows accesses which might enable running code or deploying containers in the cluster by an adversary</li><li>Examples of such interfaces that were seen exploited include Apache NiFi, Kubeflow, Argo Workflows, Weave Scope, and the Kubernetes dashboard</li></ul>
    **Vulnerable Application**<ul><li>Running an Internet-facing vulnerable application in a cluster can enable initial access to the cluster, especially if it is vulnerable to remote code execution vulnerability (RCE) that may be exploited</li></ul>
*   - 
    - Analyse collected artifacts
    - Start by [mounting the container filesystem](./playbook/analysis/mount_container_fs.md)
*   - 
    - Determine Impact
    - **Pod Escape**<ul><li>Refer to [Determine Pod Escape](./playbook/analysis/determine_pod_escape.ipynb)</li></ul>
    **Data Destruction**<ul><li>Adversary may attempt to destroy data and resources in the cluster. This includes deleting deployments, configurations, storage and compute resources</li></ul>
    **Resource Hijacking**<ul><li>Adversary may abuse a compromised resource (e.g. pod) for running tasks such as cryptocurrency mining</li><li>Adversary may also create new pods for such activities</li></ul>
    **Denial of Service**<ul><li>Includes attempts to block the availability of the pods themselves, the underlying nodes, or the API server</li></ul>
    **Exfiltration**<ul><li>Adversary may attempt to extract and steal data that is being processed or stored by cluster resources</li></ul>
*   - Containment
    - Compromised Credentials | Revoke access and/or disable/reset credentials
    - **Google Account** - Follow your company's Account Compromise playbook  
    **Access Key** - Follow your company's Key Exposure playbook  
    **kubeconfig** - [Revoke access (and refresh token)](../iam/access_tokens.html#revoke-access-tokens) in `kubeconfig`
*   - 
    - Compromised Image | Isolate the resource and take down the image
    - Refer to [Containment](./playbook/containment.md)
    <br/><br/>
    **Take down the image**<ul><li>Public registry - Request take down (e.g. [Docker Hub](https://github.com/docker/hub-feedback/issues))</li><li>Private registry (e.g. within Project) - Refer to [Container Registry Documentation](https://cloud.google.com/container-registry/docs/managing#deleting_images)</li></ul>
*   - 
    - Unintended Exposure | Close the exposure
    - Depending on the type of exposure, consider the following steps<ul><li>Rectify the ACL of the resource</li><li>Close/Filter the exposed network port</li><li>Implement secure authentication (e.g. MFA, PKI)</li></ul>
*   - 
    - Vulnerable Application | Isolate the resource and take down the application
    - Refer to [Containment](./playbook/containment.md)
*   - Remediation / Eradication
    - Remove any persistence
    - Based on persistence mechanisms identified during analysis
*   - 
    - Reset other credentials accessed by adversary
    - **Google Account** - Follow your company's Account Compromise playbook  
    **Access Key** - Follow your company's Key Exposure playbook
*   - 
    - Patch any exploited vulnerabilities
    - Download patches and apply them according to vendor’s advisories / instructions
*   - 
    - Reset unauthorised modifications
    - Based on analysis performed, and includes all impacted resources
````