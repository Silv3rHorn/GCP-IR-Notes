# Access Tokens

* Tokens can be used to authenticate to Google Cloud API
* Information about the token can be obtained via https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=<access_token>

```{image} access_tokens_1.jpg
:name: access_tokens_1
```

*   * `issued_to` 
        * If 20 digits - Internal app or associated with a service account
        * If `<12 digits>-<32 chars>.`[`apps.googleusercontent.com`](http://apps.googleusercontent.com/) - Associated with external app
    * `expires_in` - Value is in seconds
    * `access_type` - Indicates whether your application can refresh access tokens when the user is not present at the browser
        * `online` - Default value. Application cannot automatically refresh access tokens
        * `offline` - Application can automatically refresh access tokens using provided refresh token
* Lifetime of access tokens
    * By default, the maximum token lifetime is 1 hour (3,600 seconds).
    * To extend the maximum lifetime for these tokens to 12 hours (43,200 seconds), it is required to add the service account to an organization policy that includes the `constraints/iam.allowServiceAccountCredentialLifetimeExtension` list constraint
* Benefits of using access tokens in attacks
    * Access token is still valid even after removal of service account key used to generate access token

---

## Generate Access Tokens

### User Accounts

```shell
# use refresh tokens
curl --location --request POST 'https://oauth2.googleapis.com/token' \
--header 'Content-Type: application/json' \
--data-raw '{
    "client_id": "<client_id>",
    "client_secret": "<client_secret>",
    "refresh_token": "<refresh_token>",
    "grant_type": "refresh_token"
}'
```

### Service Accounts

* (Before we start) Definition
    * Base identity - Identity the adversary starts with; usually a compromised account
    * Target identity - Identity the adversary targets; can also be the base identity
* Pre-requisites
    * Either
        * Base identity has `iam.serviceAccountTokenCreator` role for Target identity 
    * OR
        * Base identity has the following project-wide permissions
            * `iam.serviceAccounts.get`
            * `iam.serviceAccounts.getAccessToken`
            * `iam.serviceAccounts.getOpenIdToken`

```shell
# set base identity as serviceAccountTokenCreator of target identity
gcloud iam service-accounts add-iam-policy-binding <target_identity> \
--member='<user_or_serviceAccount>:<base_identity>' \
--role='roles/iam.serviceAccountTokenCreator'

# [OPTIONAL] create id_token_request.json file and save in execution directory
{
    "delegates": [],
    "audience": "<base_identity>",
    "includeEmail": "true"
}

# create access_token_request.json file with the following content and save in execution directory
{
    "delegates": [],
    "scope": ["https://www.googleapis.com/auth/cloud-platform"],
    "lifetime": '3600s'
}

# [OPTIONAL] generate id token while authenticated to gcp cli as base identity
curl -X POST \
-H "Authorization: Bearer "$(gcloud auth print-access-token) \
-H "Content-Type: application/json; charset=utf-8" \
-d @id_token_request.json \
"https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/<target_identity>:generateIdToken"

# generate access token while authenticated to gcp cli as base identity
curl -X POST \
-H "Authorization: Bearer "$(gcloud auth print-access-token) \
-H "Content-Type: application/json; charset=utf-8" \
-d @access_token_request.json \
"https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/<target_identity>:generateAccessToken"

# [OPTIONAL] test access token by listing VM instances in a project
curl -X GET \
-H "Authorization: Bearer "<access_token> \
-H "Content-Type: application/json; charset=utf-8" \
"https://compute.googleapis.com/compute/v1/projects/{project}/zones/{zone}/instances"

# [OPTIONAL] test access token by deleting a VM instance
curl -X DELETE \
-H "Authorization: Bearer "<generated_access_token> \
-H "Content-Type: application/json; charset=utf-8" \
"https://compute.googleapis.com/compute/beta/projects/<project_id>/zones/<zone>/instances/<instance_name>"
```

* * *

## Revoke Access Tokens

* Obtain information about the token via https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=<access_token>
* `issued_to` is 20 digits (Internal app or service account)
    * `access_type` is `online`
        * Remove permissions/roles from associated account
        * Restore permissions/roles after expiry of access token
    * `access_type` is `offline`
        * Remove permissions/roles from associated account
        * User: Visit `google.com`, Click on Profile picture | `Manage your Google Account` | `Data & privacy` | `Third-party apps with account access` | Remove access for the internal app
        * If user is not available, a Google Workspace Admin can remove the app from user’s `Connected applications` via Google Admin console
* `issued_to` is `<12 digits>-<32 chars>.`[`apps.googleusercontent.com`](http://apps.googleusercontent.com/) (external app)
    * `curl -d -X -POST --header "Content-type:application/x-www-form-urlencoded" \
        'https://oauth2.googleapis.com/revoke?token={token}'`
        * Token can be an access token or a refresh token
        * If the token is an access token and it has a corresponding refresh token, the refresh token will also be revoked
        * If the revocation is successfully processed, then the HTTP status code of the response is `200`. For error conditions, an HTTP status code `400` is returned along with an error code