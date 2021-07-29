## argocd 네임스페이스 생성 
# kubectl create namespace argocd
## Argo CD 배포
# kubectl apply -n argocd -f https://github.com/blueice123/eks/blob/master/aggocd.yaml
## Argo CD API Server에서 외부 통신을 할 수 있도록 Argo CD의 Service의 Type을 Load Balancer로 변경(AWS에서는 Classic Load Balancer로 배포됨)
# kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
      nodeSelector:
        k8s: "admin"





# argoCD 시작
# argocd 네임스페이스 생성 
$ kubectl create namespace argocd
  
# Argo CD 배포
$ kubectl apply -n argocd -f <https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml>
# Argo CD API Server에서 외부 통신을 할 수 있도록 Argo CD의 Service의 Type을 Load Balancer로 변경(AWS에서는 Classic Load Balancer로 배포됨)
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}

# Argo CD의 pod들을 NodeSelector를 이용해서 특정 노드그룹에서만 사용될 수 있게 정의(ex, k8s:admin)
$ kubectl patch deployments argocd-dex-server -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n argocd
$ kubectl patch deployments argocd-redis -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n argocd
$ kubectl patch deployments argocd-repo-server -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n argocd
$ kubectl patch deployments argocd-server -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n argocd
$ kubectl patch statefulsets argocd-application-controller -p '{"spec": {"template": {"spec": {"nodeSelector": {"k8s": "admin"}}}}}' -n argocd


