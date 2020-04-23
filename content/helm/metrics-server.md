---
title: metrics-server
weight: 22
---

> metrics-server 을 설치 합니다.

```
cat << EOF | helm upgrade --install metrics-server stable/metrics-server --namespace kube-system --values -
args:
  # enable this if you have self-signed certificates
  - --kubelet-insecure-tls
  - --kubelet-preferred-address-types=InternalIP,InternalDNS,ExternalDNS,ExternalIP,Hostname
EOF
```
