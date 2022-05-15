# Query Language

```{tip}
Whenever possible, always define `resource.type` in the search query to speed up the search
```
```{list-table}
:header-rows: 1

*   - Query
    - Time Period
    - Approx Search Duration
    - Remarks
*   - `resource.type="gce_disk" protoPayload.methodName="v1.compute.disks.insert`
    - Last 365 days  
    31 Mar 2021 - 31 Mar 2022
    - 4 mins
    - All results returned
*   - `protoPayload.methodName="v1.compute.disks.insert`
    - Last 365 days  
    31 Mar 2021 - 31 Mar 2022
    - 26 mins (Timeout error)
    - Did not show available results before 19 Jan 2022
```

## Syntax

* Add `-` to the front of clause to negate
* Use `:` instead of `=` to do a partial string search
* Use `=~` to do a regex search
* Use `:*` to test if a field exists without testing for a particular value in the field

```
# examples
# partial match
protoPayload.methodName:"instances.insert"

# multiple OR with negation
-protoPayload.authenticationInfo.principalEmail=("user1@gmail.com" OR "user2@gmail.com" OR "user3@gmail.com")

# regex
protoPayload.requestMetadata.callerSuppliedUserAgent=~".+gcloud\.compute\.ssh.+"

# test if a field exists
protoPayload.authenticationInfo.serviceAccountKeyName:*
```