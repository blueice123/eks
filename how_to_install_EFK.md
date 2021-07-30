# 참고 문서 https://www.eksworkshop.com/intermediate/230_logging/prereqs/
로컬PC에 환경변수 정의
```bash
cluster_name=kubernetes-practice
ACCOUNT_ID=239234376445
AWS_REGION=ap-northeast-2
ES_DOMAIN_NAME="*"
```

## 만약 logging을 Fluent bit를 통해 CW logs에만 할 경우
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | sed 's/{{cluster_name}}/'${cluster_name}'/;s/{{region_name}}/'${AWS_REGION}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' | kubectl apply -f - 




## 클러스터의 서비스 계정에 IAM 역할 사용하기 위해 OIDC 자격 증명 공급자를 생성
eksctl utils associate-iam-oidc-provider \
    --cluster $cluster_name \
    --approve

## IAM policy 생성
aws iam create-policy   \
  --policy-name fluent-bit-policy \
  --policy-document file://Original_yaml/fluent-bit-policy.json

## 네임스페이스 생성
kubectl create namespace logging

## EKS에 logging 네임스페이스에 fluent-bit 서비스 계정에 대한 IAM 역할 생성
eksctl create iamserviceaccount \
    --name fluent-bit \
    --namespace logging \
    --cluster $cluster_name \
    --attach-policy-arn "arn:aws:iam::${ACCOUNT_ID}:policy/fluent-bit-policy" \
    --approve \
    --override-existing-serviceaccounts

## Annotation에 role이 잘 바인딩 되었는지 확인
kubectl -n logging describe sa fluent-bit


