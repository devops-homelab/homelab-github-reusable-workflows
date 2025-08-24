# Homelab GitHub Reusable Workflows

This repository contains enterprise-grade GitHub Actions reusable workflows designed to streamline CI/CD processes across the homelab ecosystem. These workflows support modern cloud-native applications with Gateway API, Helm charts, container security, and GitOps automation.

## Repository Structure

```
.github/
    workflows/
        helm-release.yaml        # Helm chart release automation
        helm-test.yaml          # Helm chart testing and validation
        terraform-validate.yaml # Terraform validation (planned)
        security-scan.yaml      # Container security scanning (planned)
        gateway-api-test.yaml   # Gateway API testing (planned)
```

## Workflow Features

### Production-Ready Automation
- **Semantic Versioning**: Automatic version bumping with Git tags
- **Security Scanning**: Container and chart vulnerability analysis
- **Quality Gates**: Comprehensive testing before releases
- **GitOps Integration**: Automated deployments with ArgoCD

### Cloud-Native Support
- **Gateway API**: Testing and validation workflows
- **Helm Charts**: Advanced chart management and publishing
- **Kubernetes**: Deployment validation and testing
- **Container Security**: Image scanning and policy enforcement

## Available Workflows

### `helm-release.yaml`
Automates the complete Helm chart release process with enterprise features:

**Features**:
- Automatic chart versioning based on Git tags
- Multi-environment chart publishing
- Security scanning with Trivy
- Changelog generation
- GitHub Pages deployment for chart repository
- Slack/Teams notifications

**Supported Chart Types**:
- Traditional Kubernetes manifests
- Gateway API resources
- Blue-green deployment configurations
- Multi-tenant applications

### `helm-test.yaml`
Comprehensive testing and validation for Helm charts:

**Features**:
- Helm lint with strict validation
- Chart testing with chart-testing tool
- Template rendering validation
- Gateway API resource validation
- Security policy checking
- Multi-Kubernetes version testing

**Quality Checks**:
- YAML syntax validation
- Kubernetes resource validation
- Security best practices
- Gateway API compliance
- Resource limits verification

## Usage Examples

### Basic Helm Chart Release

```yaml
name: Chart Release

on:
  push:
    branches: [main]
    tags: [v*]

jobs:
  release-chart:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-release.yaml@main
    with:
      REPO_URL: https://raw.githubusercontent.com/devops-homelab/homelab-helm-charts/gh-pages
      CHART_PATH: charts/
      ENABLE_SECURITY_SCAN: true
      NOTIFY_SLACK: true
    secrets:
      GITHUB_PAT: ${{ secrets.GH_PAT }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Comprehensive Chart Testing

```yaml
name: Chart Test

on:
  pull_request:
    types: [opened, reopened, synchronize]
    paths: [charts/**]

jobs:
  lint-test:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-test.yaml@main
    with:
      CHART_PATH: charts/
      TEST_GATEWAY_API: true
      KUBERNETES_VERSIONS: "1.26,1.27,1.28"
      ENABLE_SECURITY_SCAN: true
```

### Gateway API Validation

```yaml
name: Gateway API Validation

on:
  pull_request:
    paths: [apps/**, charts/**]

jobs:
  validate-gateway-api:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/gateway-api-test.yaml@main
    with:
      GATEWAY_CLASS: kong
      TEST_PREVIEW_ROUTES: true
      VALIDATE_TLS: true
```

## Advanced Configuration

### Multi-Environment Releases

```yaml
# Production release workflow
name: Production Release

on:
  release:
    types: [published]

jobs:
  release-production:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-release.yaml@main
    with:
      ENVIRONMENT: production
      REPO_URL: https://charts.yourdomain.com
      SIGN_CHARTS: true
      NOTARIZE_IMAGES: true
    secrets:
      COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
      HARBOR_USERNAME: ${{ secrets.HARBOR_USERNAME }}
      HARBOR_PASSWORD: ${{ secrets.HARBOR_PASSWORD }}
```

### Security-Enhanced Pipeline

```yaml
name: Security-Enhanced CI

on:
  push:
    branches: [main, develop]

jobs:
  security-scan:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/security-scan.yaml@main
    with:
      SCAN_CONTAINERS: true
      SCAN_CHARTS: true
      SCAN_TERRAFORM: true
      FAIL_ON_HIGH: true
    secrets:
      SECURITY_TOKEN: ${{ secrets.SECURITY_TOKEN }}

  test-and-release:
    needs: security-scan
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-release.yaml@main
    with:
      REQUIRE_SECURITY_APPROVAL: true
```

## Workflow Inputs

### Common Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `CHART_PATH` | Path to Helm charts | No | `charts/` |
| `REPO_URL` | Helm repository URL | Yes | - |
| `ENVIRONMENT` | Deployment environment | No | `development` |
| `ENABLE_SECURITY_SCAN` | Enable security scanning | No | `true` |

### Helm Release Specific

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `SIGN_CHARTS` | Sign charts with Cosign | No | `false` |
| `NOTARIZE_IMAGES` | Notarize container images | No | `false` |
| `AUTO_VERSION` | Automatic version bumping | No | `true` |
| `NOTIFY_SLACK` | Send Slack notifications | No | `false` |

### Helm Test Specific

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `KUBERNETES_VERSIONS` | K8s versions to test against | No | `1.28` |
| `TEST_GATEWAY_API` | Test Gateway API resources | No | `true` |
| `LINT_STRICT` | Strict Helm linting | No | `true` |
| `VALIDATE_SECURITY` | Validate security policies | No | `true` |

## Integration Examples

### ArgoCD GitOps Integration

```yaml
name: GitOps Update

on:
  workflow_run:
    workflows: ["Chart Release"]
    types: [completed]

jobs:
  update-gitops:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - name: Update ArgoCD Application
        run: |
          # Update image tags in GitOps repository
          # Trigger ArgoCD sync
```

### Multi-Repository Coordination

```yaml
name: Coordinated Release

on:
  repository_dispatch:
    types: [release-trigger]

jobs:
  release-charts:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-release.yaml@main
  
  update-infrastructure:
    needs: release-charts
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/terraform-validate.yaml@main
```

## Security Features

### Chart Signing and Verification
- Cosign integration for chart signing
- Provenance generation and verification
- SLSA compliance for supply chain security

### Container Security
- Trivy security scanning
- Policy enforcement with OPA/Gatekeeper
- Base image vulnerability assessment

### Secret Management
- GitHub encrypted secrets
- External secret injection
- Rotation and lifecycle management

## Best Practices

### Workflow Organization
- Use semantic versioning for workflow releases
- Pin workflow versions in production
- Test workflow changes in feature branches
- Document breaking changes in CHANGELOG

### Security Guidelines
- Always enable security scanning
- Use signed and verified container images
- Implement least-privilege access
- Rotate secrets regularly

### Performance Optimization
- Use workflow caching for dependencies
- Parallel job execution where possible
- Optimize Docker layer caching
- Monitor workflow execution times

## Monitoring & Observability

### Workflow Metrics
- Success/failure rates
- Execution duration
- Resource utilization
- Cost tracking

### Alerting
- Failed workflow notifications
- Security vulnerability alerts
- Performance degradation warnings
- Resource quota alerts

## Migration Guide

### From Basic to Advanced Workflows

1. **Phase 1**: Replace existing workflows with basic reusable versions
2. **Phase 2**: Enable security scanning and advanced testing
3. **Phase 3**: Implement GitOps integration and notifications
4. **Phase 4**: Add enterprise features (signing, notarization)

### Breaking Changes
- Version 2.0: Gateway API validation enabled by default
- Version 2.1: Security scanning required for production releases
- Version 2.2: Multi-environment support with enhanced secrets

## Contributing

### Development Guidelines
- Follow GitHub Actions best practices
- Include comprehensive input validation
- Add proper error handling and logging
- Test workflows in feature repositories

### Testing Process
1. Create test repository with sample charts
2. Validate workflow execution
3. Verify security scanning
4. Test failure scenarios
5. Document usage examples

### Release Process
- Semantic versioning for workflow releases
- Changelog maintenance
- Backward compatibility considerations
- Security review for all changes

## Support

### Documentation
- Workflow-specific README files
- Usage examples and tutorials
- Troubleshooting guides
- API reference documentation

### Community
- GitHub Discussions for questions
- Issue tracking for bugs and features
- Pull requests for contributions
- Regular community calls

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**Enterprise Ready • Security First • GitOps Native**
