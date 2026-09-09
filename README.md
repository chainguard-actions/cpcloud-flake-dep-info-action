# `flake-dep-info-action`

Access the fields of `flake.lock` as `outputs` in a GitHub action step.

## Inputs

```yaml
inputs:
  input:
    required: true
    description: "The flake input whose data you want to access"
  lockfile:
    required: false
    description: "The path to the flake lock file containing locked dependencies"
    default: "flake.lock"
```

## Outputs

```yaml
outputs:
  owner:
    description: "The owner of the repository"
  repo:
    description: "The repository"
  rev:
    description: "Git revision of the dependency"
  short-rev:
    description: "Short git revision"
```

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
