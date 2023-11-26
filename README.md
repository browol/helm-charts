# Browol Helm Charts

[![Chart Publish](https://github.com/browol/helm-charts/actions/workflows/release.yml/badge.svg?branch=main)](https://github.com/browol/helm-charts/actions/workflows/release.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Introduction

This repository contains a collection of Helm charts for deploying and managing various applications on Kubernetes.

## Usage

### Add Helm Repository

To install a chart from this repository, first add the repository to your Helm client:

```bash
helm repo add browol https://browol.github.io/helm-charts
helm repo update
```

Once the repository is added, you can install a chart using its name:

```bash
helm install browol/general-purpose
```

### Installing a Specific Chart Version

To install a specific version of a chart, use the `--version` flag:

```bash
helm install browol/general-purpose --version 0.2.10
```
