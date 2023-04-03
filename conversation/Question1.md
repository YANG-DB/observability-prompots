# Questions:
## finding information in logs
> **_Q:_** from all of the above text prompts, can you please tell me the indices in my system ?

> **_A:_**

    ```text
        sample-ecommerce
        .kibana_92668751_admin_1
        moviegeek-logs-2023.03.24
        .opendistro-reports-definitions
        .kibana_1
        .opendistro_security
        .opendistro-reports-instances
        otel-v1-apm-span-000
    ```
---

> **_Q:_** which of these indices are considered to have an observability logs schema mapping ?

> **_A:_** sso_logs-nginx-prod
---
> **_Q:_** can you create a ppl query to determine the average errors per day I encountered in my system ?

> **_A:_** 
```text
source=sso_logs-nginx-prod
| where log.level == 'ERROR'
| stats count() as error_count by date_histogram('@timestamp', '1d', 'UTC')
| eval days_since_start = (floor(epoch() - min('@timestamp')) / (24 * 60 * 60))
| eval avg_errors_per_day = error_count / days_since_start
```
---

> **_Q:_** can you correlate the services that are the most relevant to the errors in the former query ?

> **_A:_**
```text
source=sso_logs-nginx-prod
| where log.level == 'ERROR'
| stats count() as error_count by date_histogram('@timestamp', '1d', 'UTC'), service.name
| eval days_since_start = (floor(epoch() - min('@timestamp')) / (24 * 60 * 60))
| eval avg_errors_per_day = error_count / days_since_start
| sort by avg_errors_per_day desc
```