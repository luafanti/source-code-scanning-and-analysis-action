# GitHub workflow (action) for source code scanning & analysis

Workflow responsible for launching static code analysis, generating SBOM and scanning for vulnerabilities.
Can be applied to projects that use Gradle and Kotlin.

#### Purpose

- Static code analysis with [SonarQube](https://github.com/SonarSource/sonarqube-scan-action), [JaCoCo](https://github.com/jacoco/jacoco), and [Detekt](https://github.com/detekt/detekt)
- SBOM generation with [CycloneDX plugin](https://github.com/CycloneDX/cyclonedx-gradle-plugin) and [Trivy](https://github.com/aquasecurity/trivy)
- Vulnerabilities scan with [Trivy](https://github.com/aquasecurity/trivy)

#### Stages:

| Stage name       | Description                                                                                                                                                                                                    | Tools                   | Required |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------|----------|
| sonarqube-scan   | Performs unit tests and then calculates their coverages using the JaCoCo plugin.Finally, it sends a coverage report to the SonarQubue platform.                                                                | SonarQube, JaCoCo       | true     |
| detekt-scan      | Performs static code analysis tool for the Kotlin programming language. Produces a SARIF report which is then uploaded and viewable on GitHub Security.                                                        | Detekt                  | false    |
| cyclonedx-sbom   | Using Gradle plugin creates an aggregate of all direct and transitive dependencies of a project to produces valid CycloneDX SBOM. The job aggregates only application dependencies used in `build.gradle.kts`. | CycloneDX Gradle Plugin | false    |
| trivy-sbom       | Generates a Software Bill of Materials (SBOM) using a prebuilt jar artifact. After generation uploads SBOM with CycloneDX format as artifact and submits results to GitHub Dependency Snapshots.               | Trivy                   | true     |
| trivy-vuln       | Scans for vulnerabilities, misconfigurations, and secrets using a prebuilt jar artifact. Produces a SARIF report which is then uploaded and viewable on GitHub Security.                                       | Trivy                   | true     |

#### Inputs

| Input name              | Description                              | Type    | Required | Default |
|-------------------------|------------------------------------------|---------|----------|---------|
| detekt-scan             | Enable Kotlin scan with Detekt           | boolean | false    | true    |
| cyclonedx-sbom          | Enable CycloneDX SBOM generation         | boolean | false    | false   |
| upload-artifact         | Enable output artifacts upload           | boolean | false    | true    |
| artifact-retention-days | Uploaded artifact retention time in days | number  | false    | 7       |

#### Usage:

This workflow will be triggered when the calling workflow finished with success.

```yaml
on:
  workflow_run:
    workflows:
      - "Your calling workflow name"
    types:
      - completed

jobs:
  sbom:
    uses: luafanti/source-code-scanning-and-analysis-action/.github/workflows/workflow.yaml@main
    secrets: inherit
```

#### Notes

- `sonarqube-scan` job required gradle configuration for [SonarQube](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-gradle/) and [JaCoCo plugin](https://docs.gradle.org/current/userguide/jacoco_plugin.html)
- `cyclonedx-sbom` job required gradle configuration for [CycloneDX plugin](https://github.com/CycloneDX/cyclonedx-gradle-plugin)

