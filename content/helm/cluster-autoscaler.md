---
title: cluster-autoscaler
weight: 24
---

> 환경 변수를 설정 합니다.

```
REGION="ap-northeast-2"
CLUSTER_NAME="$(kubectl config current-context)"

ACCOUNT="$(aws sts get-caller-identity | grep "Account" | cut -d'"' -f4)"
AWS_ROLE_ARN="arn:aws:iam::${ACCOUNT}:role/${CLUSTER_NAME}-autoscaling"
```

> cluster-autoscaler 을 설치 합니다.

```
cat << EOF | helm upgrade --install cluster-autoscaler stable/cluster-autoscaler --namespace kube-system --values -
nameOverride: cluster-autoscaler

awsRegion: ${REGION}

autoDiscovery:
  enabled: true
  clusterName: ${CLUSTER_NAME}

podAnnotations:
  iam.amazonaws.com/role: ${AWS_ROLE_ARN}

extraArgs:
  v: 4
  stderrthreshold: info
  logtostderr: true
  expander: random
  scale-down-enabled: true
  skip-nodes-with-local-storage: false
  skip-nodes-with-system-pods: false

sslCertPath: /etc/ssl/certs/ca-bundle.crt

rbac:
  create: true
  pspEnabled: true
EOF
```

> 설치 내역을 확인 합니다.

```
helm list
helm history cluster-autoscaler

kubectl get pod,svc -n kube-system
```
