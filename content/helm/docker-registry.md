---
title: docker-registry
weight: 43
---

> 환경 변수를 설정 합니다.

```
# ROOT_DOMAIN="mzdev.be"
# BASE_DOMAIN="spot.${ROOT_DOMAIN}"

SERVICE_TYPE="ClusterIP"
INGRESS_ENABLED=true
INGRESS_DOMAIN="docker-registry-devops.${BASE_DOMAIN}"
```

> docker-registry 을 설치 합니다.

```
cat << EOF | helm upgrade --install docker-registry stable/docker-registry --namespace devops --values -
service:
  type: ${SERVICE_TYPE}

ingress:
  enabled: ${INGRESS_ENABLED}
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    ingress.kubernetes.io/proxy-body-size: 500m
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: 500m
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - ${INGRESS_DOMAIN}
  path: /
  tls:
    - secretName: docker-registry-tls
      hosts:
        - ${INGRESS_DOMAIN}

persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 20Gi
  storageClass: "efs"
EOF
```
