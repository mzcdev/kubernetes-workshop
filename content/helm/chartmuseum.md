---
title: chartmuseum
weight: 44
---

> 환경 변수를 설정 합니다.

```
# ROOT_DOMAIN="mzdev.be"
# BASE_DOMAIN="spot.${ROOT_DOMAIN}"

SERVICE_TYPE="ClusterIP"
INGRESS_ENABLED=true
INGRESS_DOMAIN="chartmuseum-devops.${BASE_DOMAIN}"
```

> chartmuseum 을 설치 합니다.

```
cat << EOF | helm upgrade --install chartmuseum stable/chartmuseum --namespace devops --values -
env:
  open:
    DISABLE_API: false

service:
  type: ${SERVICE_TYPE}

ingress:
  enabled: ${INGRESS_ENABLED}
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: 500m
    ingress.kubernetes.io/proxy-body-size: 500m
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - name: ${INGRESS_DOMAIN}
      path: /
      tls: true
      tlsSecret: chartmuseum-tls

persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 8Gi
  storageClass: "efs"
EOF
```
