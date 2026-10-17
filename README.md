# Client Portal BE — Jenkins UAT

Jenkins SCM for **Client Portal backend** CI/CD (UAT / dev). This repo is only the pipeline. Application code lives in [euroasia_clientportal_BE](https://github.com/ankittyagi140/euroasia_clientportal_BE).

Same pattern as [euroasia_inspection_be_jenkins_uat](https://github.com/ankittyagi140/euroasia_inspection_be_jenkins_uat).

## Jenkins job

| Setting | Value |
|---------|--------|
| Repository | `https://github.com/ankittyagi140/euroasia_clientport_BE_jenkins_UAT.git` |
| Branch | `main` |
| Script path | `Jenkinsfile` |
| Credentials | `github-pat`, `ghcr-token` |

## What the pipeline does

1. Checks out `euroasia_clientportal_BE` branch `main`
2. Lint + unit tests (`scripts/ci-lint-test.sh`)
3. Builds `ghcr.io/euroasiasci/client-portal-be:${BUILD_NUMBER}` and `:dev`
4. Pushes to GHCR
5. Triggers platform job `EUROASIA_SCI_CLIENT_PORTAL/Euroasia_Inspection/Euroasia_inspection_Infra/Euroasia_inspection_Infra` with `APPLICATION=client-portal-be`

Do **not** point this job at the Inspection BE pipeline.
