---
title: prometheus
weight: 31
---

> Nmaespace 를 생성 합니다.

```
kubectl create namespace monitor
```

> prometheus 을 설치 합니다.

```
cat << EOF | helm upgrade --install prometheus stable/prometheus --namespace monitor --values -
server:
  persistentVolume:
    enabled: true
    accessModes:
      - ReadWriteOnce
    size: 8Gi
    storageClass: "efs"

alertmanager:
  persistentVolume:
    enabled: true
    accessModes:
      - ReadWriteOnce
    size: 2Gi
    storageClass: "efs"
EOF
```

> 설치 내역을 확인 합니다.

```
helm list
helm history prometheus

kubectl get pod,svc -n monitor
```
