---
title: kube2iam
weight: 21
---

> 환경 변수를 설정 합니다.

```
REGION="ap-northeast-2"
```

> kube2iam 을 설치 합니다.

```
cat << EOF | helm upgrade --install kube2iam stable/kube2iam --namespace kube-system --values -
aws:
  region: ${REGION}
extraArgs:
  auto-discover-base-arn:
  auto-discover-default-role: true
host:
  iptables: true
  interface: eni+
rbac:
  create: true
EOF
```
