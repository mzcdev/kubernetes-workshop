---
title: sonatype-nexus
weight: 45
---

> 환경 변수를 설정 합니다.

```
PASSWORD="password"

# ROOT_DOMAIN="mzdev.be"
# BASE_DOMAIN="spot.${ROOT_DOMAIN}"

SERVICE_TYPE="ClusterIP"
INGRESS_ENABLED=true
INGRESS_DOMAIN="sonatype-nexus-devops.${BASE_DOMAIN}"
```

> sonatype-nexus 를 **StatefulSet** 으로 설치 합니다.

```
cat << EOF | helm upgrade --install sonatype-nexus stable/sonatype-nexus --namespace devops --values -
statefulset:
  enabled: true

nexus:
  service:
    type: ${SERVICE_TYPE}
  resources:
    requests:
      cpu: 1000m
      memory: 3Gi
    limits:
      cpu: 2000m
      memory: 4Gi
  livenessProbe:
    initialDelaySeconds: 100
    periodSeconds: 30
    failureThreshold: 12
    path: /
  readinessProbe:
    initialDelaySeconds: 100
    periodSeconds: 30
    failureThreshold: 12
    path: /

nexusProxy:
  env:
    nexusHttpHost: ${INGRESS_DOMAIN}

nexusBackup:
  nexusAdminPassword: ${PASSWORD}

ingress:
  enabled: ${INGRESS_ENABLED}
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    ingress.kubernetes.io/proxy-body-size: 500m
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: 500m
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  tls:
    enabled: true
    secretName: nexus-tls

persistence:
  enabled: true
  accessMode: ReadWriteOnce
  storageSize: 30Gi
  storageClass: "efs"
EOF
```
