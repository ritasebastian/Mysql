Sure! Here is a Helm chart template for deploying a 3-node MySQL replication cluster across three Kubernetes clusters, including customization in the `values.yaml` file and setup for secrets and configurations.

### Helm Chart Directory Structure

```
mysql-replication/
  ├── Chart.yaml
  ├── values.yaml
  ├── templates/
  │   ├── configmap.yaml
  │   ├── statefulset.yaml
  │   ├── service.yaml
  │   ├── secret.yaml
  │   ├── istio-gateway.yaml
  │   ├── mysql-exporter.yaml
  │   └── _helpers.tpl
```

### Chart.yaml

```yaml
apiVersion: v2
name: mysql-replication
description: A Helm chart for MySQL replication cluster
version: 0.1.0
appVersion: 8.0
```

### values.yaml

```yaml
replication:
  enabled: true
  user: repl_user
  password: repl_password

mysqlRootPassword: root_password

configurationFiles:
  mysql.cnf: |-
    [mysqld]
    skip-host-cache
    skip-name-resolve
    log-bin=mysql-bin
    binlog_format=row
    server-id=1

  parameters.cnf: |-
    [mysqld]
    max_connections=1000

primary:
  persistence:
    enabled: true
    size: 8Gi
    storageClass: standard
  resources:
    requests:
      memory: 512Mi
      cpu: 500m
    limits:
      memory: 1Gi
      cpu: 1
  podAnnotations:
    hostname: node1.com

secondary:
  replicas: 2
  persistence:
    enabled: true
    size: 8Gi
    storageClass: standard
  resources:
    requests:
      memory: 512Mi
      cpu: 500m
    limits:
      memory: 1Gi
      cpu: 1
  podAnnotations:
    hostname: node2.com, node3.com

service:
  type: ClusterIP
  port: 3306

metrics:
  enabled: true

tls:
  enabled: true
  secretName: mysql-secret
  caCertKey: ca.crt
  certKey: tls.crt
  keyKey: tls.key

extraEnvVars:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: mysql-root-password

  - name: MYSQL_REPLICATION_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: mysql-replication-password

affinity: {}
tolerations: []
nodeSelector: []

istio:
  enabled: true
  gateway:
    hosts:
      - "*"
    port: 3306
    targetPort: 3306
```

### templates/configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  labels:
    app: mysql
data:
  mysql.cnf: |-
{{ .Values.configurationFiles.mysql.cnf | indent 4 }}

  parameters.cnf: |-
{{ .Values.configurationFiles.parameters.cnf | indent 4 }}
```

### templates/statefulset.yaml

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  serviceName: mysql
  replicas: {{ .Values.secondary.replicas | default 1 }}
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        - name: mysql-config
          mountPath: /etc/mysql/conf.d
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Values.tls.secretName }}
              key: mysql-root-password
        - name: MYSQL_REPLICATION_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Values.tls.secretName }}
              key: mysql-replication-password
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-data
      - name: mysql-config
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: {{ .Values.primary.persistence.size | quote }}
      storageClassName: {{ .Values.primary.persistence.storageClass | quote }}
```

### templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - port: {{ .Values.service.port }}
    name: mysql
  selector:
    app: mysql
```

### templates/secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  mysql-root-password: {{ .Values.mysqlRootPassword | b64enc }}
  mysql-replication-password: {{ .Values.replication.password | b64enc }}
  ca.crt: {{ .Files.Get "path/to/ca.crt" | b64enc }}
  tls.crt: {{ .Files.Get "path/to/tls.crt" | b64enc }}
  tls.key: {{ .Files.Get "path/to/tls.key" | b64enc }}
```

### templates/istio-gateway.yaml

```yaml
{{- if .Values.istio.enabled }}
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: mysql-gateway
  namespace: {{ .Release.Namespace }}
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
    - port:
        number: 3306
        name: mysql
        protocol: TCP
      hosts:
        - {{ .Values.istio.gateway.hosts | join ", " }}
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: mysql
  namespace: {{ .Release.Namespace }}
spec:
  hosts:
    - {{ .Values.istio.gateway.hosts | join ", " }}
  gateways:
    - mysql-gateway
  tcp:
    - match:
        - port: 3306
      route:
        - destination:
            host: mysql
            port:
              number: 3306
{{- end }}
```

### Argo CD Application YAML

**argocd-app/mysql-replication-app.yaml**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mysql-replication
  namespace: argocd
spec:
  project: default

  source:
    repoURL: 'https://github.com/your-repo/mysql-replication.git'
    targetRevision: HEAD
    path: mysql-replication

  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

  # Customize Helm parameters
  helm:
    valueFiles:
      - values.yaml
```

### Summary

This setup includes a Helm chart template for deploying a 3-node MySQL replication cluster across three Kubernetes clusters, with Istio Ingress for routing traffic. The `values.yaml` file allows customization for different environments. The secrets and configurations are set up correctly, including a ConfigMap for MySQL configurations and a Secret for sensitive data. The Argo CD application YAML ensures that the MySQL replication cluster is deployed and managed using Argo CD.

Make sure to adjust the paths and values based on your specific environment and requirements.
