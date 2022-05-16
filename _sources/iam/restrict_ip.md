# Restrict Access by IP
During incident response, to contain/remediate an incident, there are times when it is required to block an adversary's IP address from accessing/utilising GCP resources.

## Restrict Access to GCP Cloud Console

* Requires [BeyondCorp Enterprise](https://cloud.google.com/beyondcorp-enterprise)
* [Reference](https://stackoverflow.com/questions/62698482/can-you-restrict-google-cloud-web-console-logins-to-an-ip-address-range/65989710#65989710)

## Restict Activities

* Users and service accounts are granted roles to perform activities on GCP
* Roles can optionally come with conditions (e.g. `Compute Instance Admin` role with a condition that it only applies to a specific compute instance only)
```{image} restrict_ip_1.jpg
:name: restrict_ip_1
```
* It is currently not possible to restrict roles with a condition that the activity must (not) be performed from specified IP addresses 
    * [Feature request (pending for 2 years)](https://issuetracker.google.com/issues/155582948)

## Restrict Access to Cloud Storage Bucket

* It is currently not possible to do so
* [Feature request (pending for 5 years)](https://issuetracker.google.com/issues/63068776)
