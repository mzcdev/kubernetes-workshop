---
title: grafana
weight: 32
---

> 환경 변수를 설정 합니다.

```
PASSWORD="password"

# ROOT_DOMAIN="mzdev.be"
# BASE_DOMAIN="spot.${ROOT_DOMAIN}"

SERVICE_TYPE="ClusterIP"
INGRESS_ENABLED=true
INGRESS_DOMAIN="grafana-monitor.${BASE_DOMAIN}"
```

> grafana 을 설치 합니다.

```
cat << EOF | helm upgrade --install grafana stable/grafana --namespace monitor --values -
adminUser: admin
adminPassword: ${PASSWORD}

service:
  type: ${SERVICE_TYPE}

ingress:
  enabled: ${INGRESS_ENABLED}
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - ${INGRESS_DOMAIN}
  tls:
    - secretName: grafana-tls
      hosts:
        - ${INGRESS_DOMAIN}

persistence:
  enabled: true
  accessModes:
    - ReadWriteOnce
  size: 5Gi
  storageClassName: "efs"
EOF
```

> 설치 내역을 확인 합니다.

```
helm list
helm history grafana

kubectl get pod,svc,ing -n monitor
```

> 그라파나 주소로 접속 합니다. 정의한 아이디와 패스워드로 로그인 합니다.

![Login](../../kubernetes/images/grafana_01.png)

> **Add data source** 를 선택 합니다.

![Add data source](../../kubernetes/images/grafana_02.png)

> **Prometheus** 를 선택 합니다.

![Add data source](../../kubernetes/images/grafana_03.png)

> HTTP URL 에 **http://prometheus-server** 를 입력 하고 하단에 **Save and Test** 버튼으로 Data source 를 추가 합니다.

![Add data source](../../kubernetes/images/grafana_04.png)

> 좌측의 **+** 에 마우스를 올리고, **Import** 를 선택 합니다.

![Dashboard](../../kubernetes/images/grafana_05.png)

> Garafana.com Dashboard 에 **9797** 을 입력합니다. 자동으로 load 됩니다.

![Dashboard](../../kubernetes/images/grafana_06.png)

> Prometheus 에 **Prometheus** 를 선택 합니다. **Import** 버튼으로 대시보드를 생성합니다.

![Dashboard](../../kubernetes/images/grafana_07.png)

> 클러스터 조회 대시보드가 생성 되었습니다.

![Dashboard](../../kubernetes/images/grafana_08.png)

> 같은 방법으로 **9679** 를 Import 합니다.
