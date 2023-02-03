# Charts
                           
# How to ...

## Release new chart version

Chart versioning in Chart.yaml
```yaml
version: <chart-version> # <<<< Bump version here
```

Lint the chart
```bash
helm lint ./
```

Create the Helm chart package
```bash
helm package ./ -d packages/
```

Update the Helm chart repository index
```bash
helm repo index --url ./ --merge index.yaml ./
```

Push the git repository on GitHub
```bash
git add . && \
git commit -m "your commit message" && \
git push origin main && \
git tag -f v<chart-version> $(git rev-parse --short HEAD) && \
git push --tags -f
```
