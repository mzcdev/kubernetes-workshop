---
title: efs-provisioner
weight: 25
---

> 환경 변수를 설정 합니다.

```
REGION="ap-northeast-2"
CLUSTER_NAME="$(kubectl config current-context)"
```

> 현재의 EKS cluster 용으로 미리 만들어 놓은 EFS 의 ID 를 조회 합니다.

```
EFS_ID="$(aws efs describe-file-systems --creation-token ${CLUSTER_NAME} --region ${REGION} | jq -r '.FileSystems[].FileSystemId')"
```

> efs-provisioner 을 설치 합니다.

```
cat << EOF | helm upgrade --install efs-provisioner stable/efs-provisioner --namespace kube-system --values -
efsProvisioner:
  efsFileSystemId: ${EFS_ID}
  awsRegion: ${REGION}
  path: /shared
  provisionerName: ${CLUSTER_NAME}/efs
  storageClass:
    name: efs
    isDefault: false
    gidAllocate:
      enabled: true
      gidMin: 40000
      gidMax: 50000
    reclaimPolicy: Retain
EOF
```
