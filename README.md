#  Helm Chart for General Purpose Application Deployment

## Introduction

This repository contains a collection of Helm charts for deploying and managing various applications on Kubernetes.

## Usage

### Installing a Chart

To install a chart from this repository, first add the repository to your Helm client:

```bash
helm repo add browol https://github.com/browol/helm-charts.git
```

Once the repository is added, you can install a chart using its name:

```bash
helm install browol/general-purpose
```

Replace `<chart-name>` with the name of the chart you want to install.

### Updating a Chart

To update an existing chart, first upgrade your Helm client:

```bash
helm upgrade --install browol/general-purpose
```

Replace `<chart-name>` with the name of the chart you want to update.

### Releasing a New Chart Version

1. Bump the chart version in `Chart.yaml`:

```yaml
version: <chart-version> # <<<< Bump version here
```

2. Lint the chart:

```bash
helm lint ./
```

3. Create the Helm chart package:

```bash
helm package ./ -d packages/
```

4. Update the Helm chart repository index:

```bash
helm repo index --url ./ --merge index.yaml ./
```

5. Push the changes to the git repository on GitHub:

```bash
git add . && \
git commit -m "your commit message" && \
git push origin main && \
git tag -f v<chart-version> $(git rev-parse --short HEAD) && \
git push --tags -f
```
