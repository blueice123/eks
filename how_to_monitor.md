## 로컬PC에 환경변수 정의
```bash
cluster_name=kubernetes-practice
ACCOUNT_ID=239234376445
AWS_REGION=ap-northeast-2
ES_DOMAIN_NAME="*"
```
## 참고
 - https://github.com/antonputra/tutorials/tree/main/lessons/072

# 궁금한 점..
 1. 로드밸런서를 CLB로 프로메테우스와 그라파나 각각 만들어야 하는지?
 2. 프로메테우스에서 데이터 메트릭 수집의 대상은 무엇인지? 
 3. 반드시 Postgres가 필요한지? 
 4. 그라파나 그래프를 어떻게 그려야하는지..? 
 5. kube-proxy-config의 metricsBindAddress를 0.0.0.0/0 으로 설정하면 보안상 이슈는 없는것인지? 


## local PC에 helm을 먼저 설치
brew install helm
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring \                                      
prometheus-community/kube-prometheus-stack \
--values ./Original_yaml/prometheus-values.yaml \
--version 16.10.0 \
--namespace monitoring \
--create-namespace

# 프로메테우스의 서비스를 로드밸런서로 변경
kubectl patch svc monitoring-kube-prometheus-prometheus -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'
kubectl patch svc monitoring-grafana -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'

# 관련 리소스들을 특정 노드 셀렉터를 이용하여 특정 노드 그룹으로 이동
kubectl patch deployments monitoring-grafana -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n monitoring
kubectl patch deployments monitoring-kube-prometheus-operator -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n monitoring
kubectl patch deployments monitoring-kube-state-metrics -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n monitoring
kubectl patch statefulsets alertmanager-monitoring-kube-prometheus-alertmanager -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n monitoring
kubectl patch statefulsets prometheus-monitoring-kube-prometheus-prometheus -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n monitoring

## Kube-proxy 모니터링을 위해 pod 외부에서 엑세스할 수 있도록 설정 
kubectl -n kube-system get cm kube-proxy-config -o yaml |sed 's/metricsBindAddress: 127.0.0.1:10249/metricsBindAddress: 0.0.0.0:10249/' | kubectl apply -f -

## Grafana default 패스워드
 - ID/PW: admin / test123





 
=== 아래거를 설정하면 IAM을 통해 CW logs agent로 CW logs를 보내는 설정임
 - 아래 내용 설정 전에 how_to_install_EFK.md 파일을 배포하면 amazon-cloudwatch namespace가 만들어짐

kubectl create namespace prometheus
helm install stable/prometheus --namespace prometheus --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2" --generate-name


## IAM 설정
eksctl create iamserviceaccount \
 --name cwagent-prometheus \
--namespace amazon-cloudwatch \
 --cluster $cluster_name \
--attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
--approve \
--override-existing-serviceaccounts


## 기본 구성으로 에이전트를 배포하고, 에이전트가 설치된 Amazon region으로 데이터를 보내도록 설정
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/service/cwagent-prometheus/prometheus-eks.yaml

## 에이전트가 실행 중인지 확인 
kubectl get pod -l "app=cwagent-prometheus" -n amazon-cloudwatch


====== 아래건 아닌듯??
```bash
메트릭 서버 배포 
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

단, 
Metrics Server에는 클러스터 및 네트워크 구성에 대한 특정 요구 사항이 있습니다. 이러한 요구 사항은 모든 클러스터 배포의 기본값이 아닙니다. Metrics Server를 사용하기 전에 클러스터 배포가 다음 요구 사항을 지원하는지 확인하십시오.

Metrics Server는 컨테이너 IP 주소(또는 hostNetwork가 활성화된 경우 노드 IP) 로 kube-apiserver에서 연결할 수 있어야 합니다 .
kube-apiserver는 집계 계층을 활성화 해야 합니다 .
노드에는 Webhook 인증 및 권한 부여가 활성화되어 있어야 합니다 .
Kubelet 인증서는 클러스터 인증 기관에서 서명해야 합니다(또는 --kubelet-insecure-tlsMetrics Server 에 전달 하여 인증서 유효성 검사를 비활성화해야 함 ).
컨테이너 런타임은 컨테이너 메트릭 RPC를 구현해야 합니다 (또는 cAdvisor 지원 이 있어야 함 ).