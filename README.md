# Helm Charts

[![Chart Publish](https://github.com/browol/helm-charts/actions/workflows/release.yml/badge.svg?branch=main)](https://github.com/browol/helm-charts/actions/workflows/release.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Introduction

This repository contains a collection of Helm charts for deploying and managing various applications on Kubernetes.

## Usage

### Add Helm Repository

To install a chart from this repository, first add the repository to your Helm client:

```bash
helm repo add helm-charts https://browol.github.io/helm-charts
helm repo update
```

### Installing Charts

To install, you need to specific version of the chart, use the `--version` flag:

```bash
helm install myapp helm-charts/general-purpose --version 0.3.0
```

## Configuration

```yaml
app:
  name: myapp
  image:
    registry: docker.io
    repository: nginxinc/nginx-unprivileged
    tag: latest
    pullPolicy: Always
    pullSecret: null
    containerPort: 8080
  readinessProbe:
    enabled: true
    path: /
  livenessProbe:
    enabled: true
    path: /
    initialDelaySeconds: 300

  securityContext:
    enabled: true
    spec:
      runAsUser: 1001
      runAsGroup: 1001
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true

securityContext:
  enabled: true
  spec:
    fsGroup: 1001
    supplementalGroups:
    - 1001

initContainers:
  enabled: true
  containers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://info.cern.ch
    volumeMounts:
    - name: workdir
      mountPath: /work-dir

volumeMounts:
  enabled: true
  volumes:
  - name: env-volume
    type: configMap
    mountPath: /app/dist/app/browser/assets/env/
    spec:
      name: env-from-file
      asEnvVar: ENV_FROM_FILE
      fileName: config.json
      value: |
        {
          "hello": "world"
        }
  - name: workdir
    type: emptyDir
    mountPath: /work-dir

service:
  enabled: true
  name: nginx
  ports:
  - name: proxy
    port: 80
    targetPort: 8080
    protocol: TCP

ingress:
  enabled: true
  ingresses:
  - name: my-internal-ingress
    annotations:
      nginx.ingress.kubernetes.io/service-upstream: "true"
      nginx.ingress.kubernetes.io/upstream-vhost: nginx.myabc-ns.svc.cluster.local
    ingressClassName: nginx
    paths:
    - path: /
      pathType: ImplementationSpecific
      backend:
        serviceName: nginx
        servicePort: 80
    http:
    - domainName:
      - example.com
      tls: null
  - name: my-internet-ingress
    annotations:
      nginx.ingress.kubernetes.io/service-upstream: "true"
      nginx.ingress.kubernetes.io/upstream-vhost: nginx.myabc-ns.svc.cluster.local
    ingressClassName: nginx-internet
    paths:
    - path: /
      pathType: ImplementationSpecific
      backend:
        serviceName: nginx
        servicePort: 80
    http:
    - domainName:
      - example3.com
      - example4.com

configMap:
  enabled: true
  data:
  - name: NODE_ENV
    value: "dynamic"
  - name: PRODUCTION
    value: false
  - name: USEMOCK
    value: false
  - name: AUTH_METHOD
    value: "saml"
  - name: INTERNAL_URL
    value: "http://myabc-app:8080"
  - name: PORTAL_URL
    value: "https://example.com"
  - name: LOG_LEVEL
    value: "debug"

env:
  enabled: true
  data:
  - name: MY_NODE_NAME
    valueFrom:
      fieldRef:
        fieldPath: spec.nodeName
  - name: ABC
    value: "haha"
```
