# CI Trigger Examples (main + release tags)

## Recommended production pattern
GitOps repo remains the source of truth. App repo pipelines build immutable images, then update GitOps repo values (image tags) and push/merge to `main` or a tag.

## GitHub Actions skeleton (pseudo)
```yaml
name: build-and-promote
on:
  push:
    branches: [ "main" ]
    tags: [ "v*" ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & push images
        run: |
          echo "podman/buildah build + push"
      - name: Update GitOps repo
        run: |
          echo "edit values-prod.yaml image.tag, commit, push"
```

## OpenShift Pipelines (Tekton)
Use Buildah (unprivileged) + scan + sign, then update GitOps repo.
