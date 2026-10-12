# Plan

Seven phases, at weekend pace about three to four months. Each phase ends with something that works. Check items off in the same PR as the work.

Items marked **(by hand)** are ones Thomas writes first, without Claude Code, because they are the point of the project.

## Phase 0: Repo setup

- [x] Create the public repo, add the Apache-2.0 license and a short README
- [x] Add `CLAUDE.md` and `PLAN.md`
- [x] Set up the Python project with `uv`, `ruff`, and `pytest`
- [x] Add a GitHub Actions workflow that runs lint and tests on every PR **(by hand)**
- [x] Turn on branch protection for `main`
- [x] Add `.gitignore` and `.env.example`

Done when: a PR with a failing test is blocked from merging.

## Phase 1: CLI prototype

Goal: `whyred <run-url>` prints a diagnosis for a failed GitHub Actions run.

- [ ] Fetch the failed jobs and their logs from the GitHub API
- [ ] Trim logs to the useful part (the failing step and what leads up to it)
- [ ] Fetch the diff for the commit or PR
- [ ] Redact secrets and tokens from the logs
- [ ] Write the first prompt and call the LLM with tool use **(by hand)**
- [ ] Define the diagnosis schema: cause, evidence, confidence, suggested fix
- [ ] Collect 10 real failed runs and save them as fixtures
- [ ] Unit tests against the fixtures, with no live API calls

Done when: it gives a sensible diagnosis on at least 7 of the 10 fixtures.

Learn: LLM tool use, prompt design, structured output.

## Phase 2: GitHub App

Goal: install the app on a repo and get a PR comment when a run fails.

- [ ] Register a GitHub App with the minimum permissions
- [ ] FastAPI endpoint that receives `workflow_run` webhooks
- [ ] Verify webhook signatures **(by hand)**
- [ ] Authenticate as an installation and fetch logs with the installation token
- [ ] Post the diagnosis as a PR comment, updating the old comment on re-runs
- [ ] Run it locally through a tunnel, then deploy it somewhere simple (a single container on a VPS)
- [ ] Install it on your own repos and use it

Done when: a failed run on one of your repos gets a comment within a minute or two.

Learn: webhooks, GitHub App auth, running a service.

Start showing it to people here. Ask five developers to install it.

## Phase 3: MCP and evals

Goal: the agent reaches its tools through MCP and its quality is measured.

- [ ] Write a GitHub MCP server exposing: get logs, get diff, get run history, get file **(by hand)**
- [ ] Move the agent onto MCP tools and make it a multi-step loop that decides what to look at
- [ ] Cap steps and tokens per run, and record usage and cost
- [ ] Grow the fixture set to 30 or more failures, each labeled with the real cause
- [ ] Write an eval runner that scores diagnoses against the labels
- [ ] Run the evals in CI when prompts or agent code change
- [ ] Add flaky-test detection using run history

Done when: one command reports an accuracy number, and a prompt change that makes it worse fails CI.

Learn: MCP, agent design, evaluating LLM systems.

## Phase 4: Kubernetes

Goal: the whole system runs on a cluster.

- [ ] Dockerfiles for the API and the worker **(by hand)**
- [ ] Split into a webhook receiver, a queue, and a worker
- [ ] Add Postgres for runs, diagnoses, and installations
- [ ] Run it all on a local kind cluster with raw manifests: Deployments, Services, ConfigMaps, Secrets **(by hand)**
- [ ] Add an Ingress with TLS
- [ ] Add health checks, resource requests and limits
- [ ] Convert the manifests to a Helm chart
- [ ] Stand up k3s on a cheap VPS and deploy there
- [ ] Write a read-only Kubernetes MCP server (events, pod status, logs) so the agent can diagnose deploy failures. This is the product's niche, so give it the most care.
- [ ] Break deploys on purpose (bad image tag, failing readiness probe, out-of-memory, missing secret) and add each as an eval case
- [ ] Scale the worker on queue depth

Done when: production runs on k3s, killing a worker pod loses no jobs, and the agent correctly explains each of the deliberately broken deploys.

Learn: core Kubernetes objects, networking, Helm, running stateful services.

## Phase 5: CI/CD

Goal: merging to `main` deploys to production with no manual steps.

- [ ] Build and push images in GitHub Actions, tagged by commit
- [ ] Scan images for vulnerabilities
- [ ] Install Argo CD and define the app **(by hand)**
- [ ] Deploy by updating the image tag in git
- [ ] Add a staging environment
- [ ] Add a rollback path and test it
- [ ] Run whyred on its own pipeline, including its own failed deploys

Done when: a merge reaches production in a few minutes, and a bad deploy can be rolled back with one revert.

Learn: delivery pipelines, GitOps, environments.

## Phase 6: Observability

Goal: you can tell what the system is doing and what it costs.

- [ ] OpenTelemetry tracing across the API, worker, and each agent step **(by hand)**
- [ ] Prometheus metrics: queue depth, run duration, failure rate, tokens and cost per run
- [ ] Grafana dashboards
- [ ] Structured logs with a run ID on every line
- [ ] Alerts for a stuck queue, a high error rate, and daily spend over budget

Done when: for any single diagnosis, you can pull up its trace and see each step and its cost.

Learn: tracing, metrics, alerting, cost control for LLM systems.

## Phase 7: Product

Goal: a stranger can install it and pay for it.

- [ ] Tenant isolation by installation, checked with tests
- [ ] Usage limits per plan: free for public repos, paid for private
- [ ] Billing through GitHub Marketplace or Stripe
- [ ] Landing page with a demo and install button
- [ ] Terms of service and privacy policy
- [ ] Docs for self-hosting with the Helm chart
- [ ] Launch: Show HN, relevant subreddits and Discords

Done when: someone you have never met installs it.

Learn: multi-tenancy, billing, shipping to real users.

## Open decisions

- License: Apache-2.0 for now, revisit before phase 7
- Pricing: per developer or per analyzed failure
