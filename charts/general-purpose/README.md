# Helm-standard-template
                           
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

## Usage

Add helm chart repo
```bash
helm repo add --username $GIT_USERNAME --password $GIT_PASSWORD helm-standard-template https://github.developer.allianz.io/raw/kittipol-tootong/helm-standard-template/main/
```

Run update repo
```bash
helm repo update helm-standard-template
```

Now chart is ready to run.
```bash
helm install myapp helm-standard-template/general-purpose --version <chart-version>
```

## Configuration

[values.yaml](values.yaml)

```yaml
app:
  # name: myhelloworldapp
  image:
    registry: docker.io
    repository: foobar/helloworld   # (Optional) Default: .Values.app.name | default .Release.Name
    tag: latest
    pullPolicy: Always
    
    # pullSecret: image-secret-helm
    pullSecret: null    

    # containerPort: 4200
    containerPort: null               

  readinessProbe:
    enabled: false
    path: /api/health
  livenessProbe:
    enabled: false
    path: /api/health

  # Pod annotations
  annotations: {}

  # Container-level securityContext
  securityContext:
    enabled: false
    spec:
      runAsUser: 199
      runAsGroup: 199
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true

# Pod-level securityContext
securityContext:
  enabled: false
  spec:
    fsGroup: 199
    supplementalGroups: 
      - 199 

initContainers:
  enabled: false
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
  enabled: false
  volumes:

    # Export env variable from inline data and mount as config.json file
    - name: env-volume
      type: configMap
      mountPath: /app/dist/app/browser/assets/env/
      spec: 
        name: env-from-file
        asEnvVar: ENV_FROM_FILE # (Optional) Environment variable
        fileName: config.json
        value: |
          {
            "hello": "world"
          }

    # Export env variable from json file and mount as config.json file
    - name: json-from-file-as-env
      type: configMap
      mountPath: /app/assets/
      spec: 
        name: env-from-file           # Configmap name
        asEnvVar: MY_CONFIG           # (Optional) Environment variable
        fileName: config.json         # Filename in configmap
        fromFile: ./config.json       # Path to your json file

    # mounting persistance volume claim into pods from azure disk
    - name: my-azure-managed-disk
      type: azureDisk
      mountPath: /tmp/data/
      spec:
        diskName: pvcRestored
        diskURI: /subscriptions/19da35d3-9a1a-4f3b-9b9c-3c56ef409565/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/pvcRestored

    # mounting persistance volume claim into pods from existing pvc
    - name: my-existing-pvc
      type: persistentVolumeClaim
      mountPath: /tmp/app/
      spec:
        claimName: azure-default-disk

    # mounting emptyDir volume
    - name: workdir
      type: emptyDir
      mountPath: /work-dir

service:
  enabled: false
  name: angular
  ports:
    - name: proxy
      port: 8080
      targetPort: 4200
      protocol: TCP

ingress:
  enabled: false
  ingresses:
    - name: my-internal-ingress
      annotations:
        nginx.ingress.kubernetes.io/service-upstream: "true"
        nginx.ingress.kubernetes.io/upstream-vhost: angular.myabc-ns.svc.cluster.local
      ingressClassName: nginx 
      paths:
        - path: /
          pathType: ImplementationSpecific # this is default value
          backend:
            serviceName: angular
            servicePort: 8080
      http:
        - domainName: 
            - example.com
          tls: null
        - domainName: 
            - example2.com
          tls:
            existingSecret: null
            cert:
              name: pubcert
              path: certificates/pub # path to certificate directory
              fileName: "*"
    - name: my-internet-ingress
      annotations:
        nginx.ingress.kubernetes.io/service-upstream: "true"
        nginx.ingress.kubernetes.io/upstream-vhost: angular.myabc-ns.svc.cluster.local
      ingressClassName: nginx-internet
      paths:
        - path: /
          pathType: ImplementationSpecific # this is default value
          backend:
            serviceName: angular
            servicePort: 8080
      http:
        - domainName: 
            - example3.com
            - example4.com
          tls:
            existingSecret: my-tls-secret
        - domainName: 
            - example5.com
          tls:
            existingSecret: null
            cert:
              name: extcert
              path: certificates/ext # path to certificate directory
              fileName: "*"
      defaultBackend:
        service:
          name: defaultbackend
          port:
            number: 8080
  
ConfigMap:
  enabled: false
  data:
    - name: NODE_ENV
      value: "dynamic"
    - name: PRODUCTION
      value: false
    - name: USEMOCK
      value: false
    - name: AUTH_METHOD
      value: "saml"
    - name: AF_API_INTERNAL_URL
      value: "http://myabc-app:8080"
    - name: AF_PORTAL_URL
      value: "https://example.com"
    - name: LOG_LEVEL
      value: "debug"

envFrom:
  enabled: false
  
# envFrom:
#   enabled: false
#   data:
#     - configMapRef:
#           name: env-proxy

env:
  enabled: false
  data:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: ABC
      value: "haha"

# key Vault for public cloud provider
keyVault:
  enabled: false
  config:
    autoVolume: true
    provider: azure
    driver: secrets-store.csi.k8s.io
    resourceGroup: rg-example
    subscriptionId: example-subscription-id
    tenantId: 6e06e42d-6925-47c6-b9e7-9581c7ca302a
    secretRef: secrets-store-creds-helm
    cloudName: AzurePublicCloud
    vaultName: kv-allianz-for-dev-ub79
  objects:
    - secretName: akssecret
      type: Opaque
      data:
        - name: session-secret
          key: SESSION_SECRET

resources: {}
```

For more please go to [examples directory](/examples/)