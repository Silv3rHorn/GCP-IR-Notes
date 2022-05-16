# VM Instances for IR
````{div} full-width
```{list-table}
:header-rows: 1

*   - Purpose
    - Recommended Specs
    - Recommended Software
    - Approx Cost (USD)/month
    - Availability
*   - Triage and GCP investigation
    - **Machine Type**: `e2-medium` (2vCPUs, 4GB RAM)  
      **Boot Disk**: `30 GB Standard Persistent Disk`  
      **Service Account**: Account referenced [here](svc_acct.md)  
      **Cloud API access scopes**: `Allow full access to all Cloud APIs`  
      **OS**: `Ubuntu 20.04 LTS`
    - [Ops Agent](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent)  
      jq  
      [kubectl](https://kubernetes.io/docs/tasks/tools/)
      <br></br>
      **Python libraries**:  
      pandas  
      ipywidgets
    - 33.47
    - Always started
*   - Forensic and malware analysis (where created forensic disk is attached to)
    - **Machine Type**: `e2-standard-4` (4vCPUs, 16GB RAM)  
      **Boot Disk**: `128 GB SSD Persistent Disk`  
      **Network**: Isolated network from other compute instances  
      **Service Account**: `Compute Engine default service account`  
      **Cloud API access scopes**: `Allow default access`  
      **OS**: `Ubuntu 20.04 LTS`
    - [Ops Agent](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent)  
      [SIFT](https://github.com/teamdfir/sift-cli)  
      Go  
      jq  
      [Container Explorer](https://github.com/google/container-explorer)    
      [Docker Explorer](https://github.com/google/docker-explorer)
    - 144.63 (if always started)
    - Started when needed
```
````