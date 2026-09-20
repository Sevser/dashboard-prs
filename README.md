# GitHub PR Dashboard

A local-first dashboard for understanding how pull requests move from creation through review to merge. The goal is to make PR lead time, review delays, throughput, and aging work easy to explore across selected GitHub repositories.

## Current status

The local Apache DevLake Docker Compose environment is configured. The custom application has not been implemented, and no GitHub repositories are connected yet.

## Planned architecture

```text
GitHub repositories
        |
        v
Apache DevLake (scheduled collection)
        |
        v
MySQL (collected and normalized data)
        |
        v
Application backend (read-only queries and metric calculations)
        |
        v
Custom dashboard
```

Apache DevLake will run locally using Docker Compose. Its configuration UI will manage GitHub connections and collection schedules. The bundled Grafana dashboards will help us inspect the imported data and validate metrics before building the custom dashboard.

The application framework, backend stack, and charting library are still to be chosen. The browser will access data through our backend; database credentials and GitHub tokens stay server-side.

## Initial dashboard scope

- Filter by repository and date range.
- Show PR open-to-merge time, time to first review, and weekly merged PR counts.
- List open PRs by age, with links to GitHub.
- Show trends and a PR table so aggregate metrics can be traced back to individual PRs.
- Display collection status and data freshness.

### Proposed metric definitions

| Metric | Initial definition |
| --- | --- |
| PR lead time (open to merge) | Elapsed time between PR creation and merge, for PRs merged within the selected period. |
| Time to first review | Elapsed time between PR creation and the first submitted review by someone other than the PR author, for PRs created within the selected period. PRs without a qualifying review have no value. |
| Merge throughput | Number of PRs merged per week, based on merge time. |
| Open PR age | Elapsed time from creation to the current time for currently open PRs, including drafts. This is a current snapshot, independent of the historical date filter. |

Start with median and 75th-percentile duration summaries plus sample counts. Missing timestamps must remain missing rather than becoming zero. Initial elapsed-time metrics include weekends and time spent in draft. Bot reviews are included initially; any later exclusions must be explicit.

These definitions are proposals to validate against the available DevLake data. PR open-to-merge time is distinct from DORA lead time for changes, which requires deployment context.

## Implementation plan

1. Connect one GitHub repository through the DevLake configuration UI and import a limited historical range.
2. Inspect the collected schema and verify the proposed metrics against real PRs.
3. Choose the application stack and implement a backend with read-only database access.
4. Build the dashboard filters, metric cards, trends, and PR table.
5. Validate calculations and document local development commands.

## Local setup

Docker Desktop with Docker Compose is required. The configuration pins Apache DevLake to `v1.0.3-beta17`, stores MySQL and Grafana data in named volumes, and only exposes the browser interfaces on the local machine.

Create your private environment file and replace every placeholder with a unique value:

```powershell
Copy-Item .env.example .env
```

`ENCRYPTION_SECRET` must contain exactly 128 uppercase ASCII letters. Keep it safe and stable: DevLake uses it to encrypt stored GitHub tokens and other credentials.

Start the services:

```powershell
docker compose up -d
```

Then open:

- DevLake configuration: <http://localhost:4000>
- Grafana dashboards: <http://localhost:3002>

Check service state and logs with:

```powershell
docker compose ps
docker compose logs -f devlake
```

Stop the services without deleting collected data:

```powershell
docker compose down
```

Do not add `--volumes` unless you intentionally want to erase the local DevLake database and Grafana state.

Reference documentation:

- [DevLake Docker Compose installation](https://devlake.apache.org/docs/GettingStarted/DockerComposeSetup/)
- [DevLake GitHub integration](https://devlake.apache.org/docs/Configuration/GitHub/)
- [DevLake releases](https://github.com/apache/devlake/releases)

Keep GitHub tokens, encryption secrets, and database credentials out of version control. Configure credentials locally and retain the DevLake encryption secret alongside the persistent instance.

## Out of scope for the first version

- Modifying or merging pull requests.
- Multi-tenant hosting and public deployment.
- Individual developer rankings.
- Deployment-based DORA metrics.
