helm repo add n8n https://helm.n8n.io
helm repo update
helm search repo n8n/n8n --versions



argocd app create n8n \
  --repo https://charts.bitnami.com/bitnami \
  --helm-chart n8n \
  --revision 1.22.0 \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace n8n \
  --sync-policy automated \
  --helm-set image.tag=1.28.0 \
  --helm-set service.type=LoadBalancer \
  --helm-set service.loadBalancerIP=10.0.0.83 \
  --helm-set persistence.enabled=true \
  --helm-set persistence.size=2Gi \
  --helm-set postgresql.enabled=true \
  --helm-set postgresql.primary.persistence.size=3Gi \
  --helm-set postgresql.auth.database=n8n \
  --helm-set postgresql.auth.username=n8n \
  --helm-set postgresql.auth.password='kaLIst01!' \
  --helm-set postgresql.auth.postgresPassword='kaLIst01!' \
  --helm-set resources.requests.memory=256Mi \
  --helm-set resources.requests.cpu=100m \
  --helm-set resources.limits.memory=512Mi
