# Client Portal BE — Jenkins pipelines

Jenkins SCM for **Client Portal backend** CI/CD. Application code lives in [euroasia_clientportal_BE](https://github.com/ankittyagi140/euroasia_clientportal_BE).

| Job | Script path | App branch | Image tags | Platform job |
|-----|-------------|------------|------------|--------------|
| UAT / dev | `Jenkinsfile` | `dev` | `:BUILD_NUMBER` and `:dev` | `…/Euroasia_inspection_Infra` |
| Production | `Jenkinsfile.prod` | `release` | `:BUILD_NUMBER` and `:prod` | `…/Euroasia_inspection_Infra_PRODUCTION` |

Credentials: `github-pat`, `ghcr-token`.

Promote to production by merging `dev` → `release` on the **app** repo, then run the prod Jenkins job.

Do **not** point these jobs at the Inspection BE pipeline.
