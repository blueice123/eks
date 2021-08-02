<!-- 참고: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html -->

## 로컬PC에 환경변수 정의
```bash
cluster_name=kubernetes-practice
ACCOUNT_ID=239234376445
AWS_REGION=ap-northeast-2
```


aws eks update-cluster-config \
    --region $AWS_REGION \
    --name $cluster_name \
    --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enabled":true}]}'



## CloudWatch의 네임스페이스를 생성하려면
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml

## CloudWatch 에이전트에 대한 서비스 계정을 생성하려면
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-serviceaccount.yaml

## CloudWatch 에이전트에 대한 ConfigMap을 생성하려면
 - 구성시 클러스터 이름을 변경할것!! 
kubectl apply -f ./Original_yaml/cwagent-configmap.yaml

원본: 
curl -O https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-configmap.yaml


## CloudWatch 에이전트를 DaemonSet으로 배포하려면
kubectl apply -f ./Original_yaml/cwagent-daemonset.yaml
