# Create Alerts

GCP allows for the creation of alerts based on specified log-based criteria, but it is not obvious how it can be done.

The process consists of 4 main steps:
- [Add / Edit Notification Channels](https://console.cloud.google.com/monitoring/alerting/notifications)
- [Create Logs-based Metric](#create-logs-based-metric)
- [Create Alerting Policy](#create-alerting-policy)
- Test the created alert

## Create Logs-based Metric
1. Access https://console.cloud.google.com/logs/metrics
2. Click on `CREATE METRIC` under User-defined Metrics
```{image} create_alerts_1.jpg
:name: create_alerts_1
```
3. Provide the requested information
    - **Metric Type**: `Counter`
    - **Details**
        - **Log metric name**: `<name>`
        - **Description**: `<desciption>`
        - **Units**: `1`
    - **Filter selection**: `<log-based query criteria>`
    - **Labels**: Ignore
4. Click `CREATE METRIC`

## Create Alerting Policy
1. Access https://console.cloud.google.com/monitoring/alerting
2. Click on `Create Policy` at the menu bar
```{image} create_alerts_2.jpg
:name: create_alerts_2
```
3. Click `SELECT A METRIC`
4. Disable `Show only active resources & metrics`
5. Search for and select `logging/user/<log_metric_name_configured_above>`
6. Provided the requested information:
    - Transform data
        - Rolling window: `1 min`
        - Rolling window function: `count`
7. Click on `Next`
8. Provide the requested information
    - Condition type: `Threshold`
    - Alert trigger: `Any time series violates`
    - Threshold position: `Above threshold`
    - Threshold value: `0.99`
9. Click on `Next`
10. Select previously configured notification channel and preferred incident autoclose duration
11. Give the alert a name
12. Click on `Next`
13. Click on `CREATE POLICY`