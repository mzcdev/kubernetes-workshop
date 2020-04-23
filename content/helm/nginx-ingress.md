---
title: nginx-ingress
weight: 12
---

> Nmaespace 를 생성 합니다.

```
kubectl create namespace kube-ingress
```

> nginx-ingress 를 **DaemonSet** 으로 설치 합니다.

```
cat << EOF | helm upgrade --install nginx-ingress stable/nginx-ingress --namespace kube-ingress --values -
controller:
  kind: DaemonSet
  config:
    use-forwarded-headers: "true"
  service:
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
EOF
```

> 설치 내역을 확인 합니다.

```
helm list
helm history nginx-ingress

kubectl get pod,svc -n kube-ingress
```

> ELB 주소를 확인 합니다.

```
ELB_DOMAIN="$(kubectl get svc -n kube-ingress | grep LoadBalancer | grep nginx-ingress-controller | awk '{print $4}')"
```

> ELB 주소를 Route53 에서 원하는 도메인의 CNAME 으로 등록 합니다.

> **mzdev.be** 라는 도메인이 있고, ELB 주소를 ***.spot.mzdev.be** 로 연결 하고자 할때 다음과 같은 방법으로 등록 할수 있습니다.

```
ROOT_DOMAIN="mzdev.be"
BASE_DOMAIN="spot.${ROOT_DOMAIN}"

# HostedZoneId
ZONE_ID="$(aws route53 list-hosted-zones | ROOT_DOMAIN=${ROOT_DOMAIN}. jq -r '.HostedZones[] | select(.Name==env.ROOT_DOMAIN) | .Id' | cut -d'/' -f3)"

# CanonicalHostedZoneNameID
ELB_SUB_ID="$(echo ${ELB_DOMAIN} | cut -d'-' -f1)"

ELB_ZONE_ID="$(aws elb describe-load-balancers --load-balancer-name ${ELB_SUB_ID} | jq -r '.LoadBalancerDescriptions[] | .CanonicalHostedZoneNameID')"
```

```
RECORD="/tmp/route53_record.json"

cat > ${RECORD} << EOF
{
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "*.${BASE_DOMAIN}",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "${ELB_ZONE_ID}",
          "DNSName": "${ELB_DOMAIN}",
          "EvaluateTargetHealth": false
        }
      }
    }
  ]
}
EOF

aws route53 change-resource-record-sets --hosted-zone-id ${ZONE_ID} --change-batch file://${RECORD}
```
