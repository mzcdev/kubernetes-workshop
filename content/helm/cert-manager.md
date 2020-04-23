---
title: cert-manager
weight: 13
---

> CustomResourceDefinition 을 생성 합니다.

```
kubectl apply \
    --validate=false \
    --filename=https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
```

> jetstack Repository 를 등록 합니다.

```
helm repo add jetstack https://charts.jetstack.io

helm repo update
```

> cert-manager 를 설치 합니다.

```
helm upgrade --install cert-manager jetstack/cert-manager \
    --namespace kube-ingress \
    --version v0.11.0
```

> 설치 내역을 확인 합니다.

```
helm list
helm history cert-manager

kubectl get pod,svc -n kube-ingress
```

> ClusterIssuer 를 설치 합니다. 이때 도메인 주인의 email 을 입력 하도록 합니다.

```
EMAIL="me@nalbam.com"

cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: ${EMAIL}
    privateKeySecretRef:
      name: letsencrypt-prod
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    # An empty 'selector' means that this solver matches all domains
    - selector: {}
      http01:
        ingress:
          class: nginx
EOF
```
