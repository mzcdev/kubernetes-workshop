---
title: jenkins
weight: 41
---

> 환경 변수를 설정 합니다.

```
PASSWORD="password"

# ROOT_DOMAIN="mzdev.be"
# BASE_DOMAIN="spot.${ROOT_DOMAIN}"

SERVICE_TYPE="ClusterIP"
INGRESS_ENABLED=true
INGRESS_DOMAIN="jenkins-devops.${BASE_DOMAIN}"
```

> Nmaespace 를 생성 합니다.

```
kubectl create namespace devops
```

> jenkins 을 설치 합니다.

```
cat << EOF | helm upgrade --install jenkins stable/jenkins --namespace devops --values -
master:
  adminUser: admin
  adminPassword: ${PASSWORD}
  resources:
    requests:
      cpu: 1000m
      memory: 3Gi
    limits:
      cpu: 2000m
      memory: 4Gi
  hostNetworking: true
  javaOpts: "-Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Seoul"
  serviceType: ${SERVICE_TYPE}
  ingress:
    enabled: ${INGRESS_ENABLED}
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
    hostName: ${INGRESS_DOMAIN}
    tls:
      - secretName: jenkins-tls
        hosts:
          - ${INGRESS_DOMAIN}
  overwritePlugins: true
  installPlugins:
    - kubernetes:latest
    - workflow-job:latest
    - workflow-aggregator:latest
    - credentials-binding:latest
    - blueocean:latest
    - kubernetes-credentials-provider:latest
    - pipeline-github-lib:latest
    - active-directory:latest
    - role-strategy:latest
    - ldap:latest
    - google-login:latest
  prometheus:
    enabled: false

persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 8Gi
  storageClass: "efs"
EOF
```
