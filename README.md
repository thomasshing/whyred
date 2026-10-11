# whyred

"Why is the build red." An agent that diagnoses failed CI runs and comments the likely cause on the pull request.

Its focus is Kubernetes deploy failures: explaining why a rollout failed by reading cluster events, pod status, and logs alongside the CI logs.

## Status

Early. Nothing is runnable yet. The project is at phase 0 (repo setup) of the roadmap in [PLAN.md](PLAN.md).

## How it will work

1. A GitHub Actions run fails.
2. GitHub sends a `workflow_run` webhook to the service.
3. The agent pulls the failed job logs, the diff, and recent run history.
4. It posts a comment on the PR with the diagnosis, the evidence, and a suggested fix.

For deploy failures, the agent also reads cluster state with read-only access. It never gets write access to a cluster, and its only write action on GitHub is its own comment.

## Stack

Python 3.12, FastAPI, the Claude API, MCP servers for GitHub and Kubernetes, Postgres, Redis, Helm, Argo CD, and OpenTelemetry with Prometheus and Grafana.

## Why

This is a learning project first and a product second, built to understand Kubernetes, agentic AI, MCP, CI/CD, and observability in depth.

## License

[Apache-2.0](LICENSE)
