## Tech Stack

- **Java 21** — language version
- **Spring Boot 3.4.4** with `spring-boot-starter-web` + `spring-boot-starter-actuator`
- **Maven** — build tool
- **Docker** — multi-stage build with BuildKit cache mounts
- **Helm v2** — Kubernetes packaging (Deployment + Service ClusterIP + Ingress, no HPA)
- **GitHub Actions** — CI pipeline: Maven build, Docker build (with GHA cache), Helm lint + publish automático no push para `main` (imagem + chart no GHCR, versão `1.0.<run_number>`)

## Project Structure

```
my-java-app2/
├── pom.xml                              # Java 21, Spring Boot 3.4.4
├── AGENTS.md                            # this file
├── .gitignore                           # target/, .idea/, *.iml, *.swp
├── src/main/java/com/example/
│   ├── Application.java                 # @SpringBootApplication
│   └── HelloController.java             # GET /hello → "Hello from my-java-app2!"
├── src/main/resources/
│   └── application.yml                  # server.port=8080, actuator /health
├── docker/
│   └── Dockerfile                       # multi-stage (maven build → jre runtime)
├── local/
│   ├── build.sh                         # mvn clean package
│   ├── run.sh                           # mvn spring-boot:run
│   ├── docker.sh                        # docker build + run
│   ├── helm-validate.sh                 # helm lint + template
│   ├── k8s-deploy.sh                    # build → docker → kind load → helm install
│   ├── k8s-test.sh                      # port-forward + curl /hello + /health
│   ├── k8s-clean.sh                     # helm uninstall
│   ├── publish-image.sh                 # docker build + tag + push to GHCR
│   ├── publish-chart.sh                 # helm package + push to GHCR OCI
│   └── publish-all.sh                   # build → publish-image → publish-chart
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── .helmignore
│   └── templates/
│       ├── _helpers.tpl
│       ├── deployment.yaml              # liveness+readiness via /actuator/health
│       ├── ingress.yaml                 # Ingress (nginx, controlado por values.ingress.enabled)
│       └── service.yaml                 # ClusterIP port 8080
└── .github/
    └── workflows/
        └── ci.yml                       # GHA: Maven build, Docker build (GHA cache), Helm lint, publish (main)