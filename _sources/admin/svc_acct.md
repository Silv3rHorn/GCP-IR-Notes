# Service Account

In an enterprise GCP environment, there is typically one or more GCP Organisations with multiple Folders and Projects under it. There could also be orphan projects (i.e. projects with no parent organisation) used by some teams.

To ensure that incident response could be performed effectively across all the aforementioned projects, a service account should be created in the **Analyst** project with the necessary role / permissions granted at the Organisation level or each individual project level. (GCP allows a service account created at a separate Organisation / Project to be added to another Organisation's / Project's IAM, and assigned role(s))

Such a service account is expected to be already granted **Full Viewer Access** (read permissions) to the entire Organisation / Projects using the following roles - `Organization Viewer`, `Folder Viewer`, `Viewer`, `Private Logs Viewer`. Hence, this section would focus on the minimum **NON-READ** permissions that should be granted to the service account, and how the account can be hardened to prevent unauthorised usage of these permissions.

```{tableofcontents}
```
