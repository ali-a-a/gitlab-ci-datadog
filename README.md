# Gitlab CI Observability in Datadog

Monitoring solution for GitLab CI pipelines using [mvisonneau/gitlab-ci-pipelines-exporter](https://github.com/mvisonneau/gitlab-ci-pipelines-exporter) and Datadog. This project provides a ready-to-use Datadog dashboard and a Helm configuration for exporting GitLab CI metrics.

## Overview

- **Exporter**: Collects GitLab CI pipeline and job metrics and exposes them via Prometheus/OpenMetrics.
- **Datadog Agent**: Scrapes the exporter metrics and sends them to Datadog.
- **Dashboard**: Visualizes pipeline and job health, durations, queue times, artifact sizes, and more.

---

## Features

- **Pipeline Success Rate** (weekly)
- **Queued Tasks, Projects, and Refs** counts
- **Average Pipeline and Job Durations**
- **Top 10 Longest Pipelines/Jobs**
- **Failed Jobs by Project and Stage**
- **Job Run Counts (last 24h)**
- **Artifact Size per Job**
- **Average Job Queue Time**
- **Top Jobs with Longest Queue Times**

All widgets are defined in `gitlab-ci-datadog-dashboard.json` and use template variables for project, ref, and stage.

---

## Quick Start

### 1. Prerequisites

- Kubernetes cluster
- Helm
- Datadog Agent deployed in your cluster
- GitLab personal access token

### 2. Configuration

Edit `helm/values.yaml` to match your environment:

- **podAnnotations**: Configures the Datadog Agent to scrape the exporter pod using OpenMetrics.
- **redis**: By default, disables Bitnami Redis. You can point to your own Redis instance for HA.
- **config.gitlab**: Set your GitLab URL and token.
- **config.projects**: List the GitLab projects to monitor.
- **config.project_defaults**: Controls which refs, tags, and jobs are monitored.

Example:
```yaml
config:
  redis:
    url: redis://your-redis-host:6379
  gitlab:
    url: https://gitlab.com
    token: {{ requiredEnv "GCPE_GITLAB_TOKEN" | quote }}
  projects:
    - name: your_gitlab_group/your_project
  project_defaults:
    output_sparse_status_metrics: true
    pull:
      refs:
        branches:
          regexp: "^(?:main|master)$"
        tags:
          regexp: ".*"
          most_recent: 3
        merge_requests:
          enabled: false
      pipeline:
        jobs:
          enabled: true
```

### 3. Deploy

```sh
helm install gitlab-ci-pipelines-exporter ./helm -n monitoring
```

---

## Datadog Dashboard

Import `gitlab-ci-datadog-dashboard.json` into Datadog to get a comprehensive view of your GitLab CI metrics.

### Main Widgets

- **Pipeline Success Rate**: Shows the percentage of successful pipelines weekly.
- **Queued Tasks/Projects/Refs**: Current counts.
- **Average Durations**: For pipelines and jobs.
- **Toplists**: Longest pipelines/jobs and jobs with longest queue times.
- **Failed Jobs Table**: Breakdown by project and stage.
- **Artifact Size**: MB per job.
- **Job Run Count**: Bar chart for last 24h.

All widgets use template variables for filtering by project, ref, and stage. A preview of the dashboard:

![422437777-0903b96f-56e9-4396-bf1e-867ec53acd70](https://github.com/user-attachments/assets/def795ba-e149-4d7e-ac6c-91f047167d20)
![422437777-0903b96f-56e9-4396-bf1e-867ec53acd70](https://github.com/user-attachments/assets/53648581-818e-4794-85cb-536dcf851c16)

---

## References

- [gitlab-ci-pipelines-exporter](https://github.com/mvisonneau/gitlab-ci-pipelines-exporter)
- [Datadog OpenMetrics Integration](https://docs.datadoghq.com/integrations/openmetrics/)
- [Helm](https://helm.sh/)

---

## License

Apache 2.0. See [LICENSE](LICENSE) for details.
