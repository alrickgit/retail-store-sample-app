<!-- Guidance for AI coding agents working on this repository -->
# Copilot instructions — Retail Store Sample App

Purpose: give an AI agent the minimum, actionable context to be productive in this monorepo.

- **Big picture**
  - Multi-service retail sample deployed via GitOps (ArgoCD) to EKS. Services live under `src/`:
    - `src/ui/` (Java, Helm chart)
    - `src/catalog/` (Go, HTTP API, `main.go`, Helm chart)
    - `src/cart/`, `src/orders/` (Java, Spring Boot)
    - `src/checkout/` (Node/NestJS)
  - Infrastructure in `terraform/` provisions EKS, ArgoCD, ingress, cert-manager and ECR.
  - Each service has a `chart/` subfolder (Helm) and a `Dockerfile` for image build.

- **Service conventions & patterns**
  - Java services use Maven wrapper: prefer `./mvnw` (Linux/mac) or `mvnw.cmd` (Windows).
  - Java projects use Spring Boot, Java 21, MapStruct and Lombok; checkstyle is enforced via the POM.
  - Catalog (Go) exposes `/health` and `/topology`; it uses a config struct (`config` package) to determine persistence provider (`in-memory`, `mysql`, etc.).
  - Checkout (Node) is NestJS; scripts are in `package.json` (`serve:dev`, `build`, `test:integration`).
  - Prometheus and OpenTelemetry are wired into services (OTEL enabled when `OTEL_SERVICE_NAME` is present).
  - Tests commonly use Testcontainers (Java/Node) and `go test` for Go code.

- **Common local run/build commands** (examples)
  - Terraform infra: `cd terraform && terraform init && terraform apply --auto-approve`
  - Java service (Linux/mac): `cd src/cart && ./mvnw spring-boot:run` or `./mvnw package && java -jar target/*.jar`
  - Java service (Windows): use `mvnw.cmd` instead of `./mvnw`.
  - Go catalog: `cd src/catalog && go run main.go` (or `go build && ./catalog`)
  - Node checkout: `cd src/checkout && npm install && npm run serve:dev` (or `npm run build`)
  - Run Java unit tests: `cd src/cart && ./mvnw test` (integration tests may require Testcontainers)

- **Where to look for patterns and examples**
  - Helm values and chart examples: each service `chart/values.yaml` (e.g. [src/catalog/chart/values.yaml](src/catalog/chart/values.yaml)).
  - Infra and GitOps wiring: `terraform/` + `argocd/applications/*.yaml` (ArgoCD app definitions reference the Helm charts).
  - Service entrypoints and middleware:
    - `src/catalog/main.go` — shows OTEL init, Prometheus, chaos middleware, `/topology` and `/health` endpoints.
    - `src/cart/pom.xml` — demonstrates BOMs, checkstyle plugin and Testcontainers usage.
    - `src/checkout/package.json` — lists dev/test scripts and Jest configuration (integration tests under `test/`).

- **Integration notes & pitfalls**
  - Production workflow expects images pushed to ECR and ArgoCD to deploy from charts; changing chart values can alter runtime topology.
  - Many runtime behaviors are controlled by environment variables (DB type, OTEL config). Check `config` packages in each service.
  - Health checks and chaos middleware can cause intermittent 503s locally; use `/health` to verify readiness.
  - Some README examples assume AWS root credentials and public ECR images — do not hardcode credentials in PRs.

- **When you modify code**
  - Keep service/Helm chart parity: if you add env var usage, update `chart/values.yaml` and `templates/deployment.yaml`.
  - For Java changes, maintain `pom.xml` dependency management (BOM entries) and run `./mvnw package` to verify.
  - For infra changes, update `terraform/` and `argocd/applications/*.yaml` consistently.

If anything here is unclear or you want deeper examples (CI pipelines, image build steps, or a service walkthrough), tell me which area to expand.
