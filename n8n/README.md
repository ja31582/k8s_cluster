helm repo add n8n https://helm.n8n.io
helm repo update
helm search repo n8n/n8n --versions



argocd app create n8n \
  --repo https://github.com/n8n-io/n8n-helm.git \
  --path n8n \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace n8n \
  --sync-policy automated \
  --helm-set persistence.enabled=true \
  --helm-set persistence.size=2Gi \
  --helm-set postgresql.enabled=true \
  --helm-set postgresql.primary.persistence.size=3Gi \
  --helm-set postgresql.auth.database=n8n \
  --helm-set postgresql.auth.username=n8n \
  --helm-set postgresql.auth.password='kaLIst01!' \
  --helm-set service.type=LoadBalancer \
  --helm-set service.loadBalancerIP=10.0.0.83


argocd app create n8n \
  --repo https://community-charts.github.io/helm-charts \
  --helm-chart n8n \
  --revision latest \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace n8n \
  --sync-policy automated \
  --helm-set persistence.enabled=true \
  --helm-set persistence.size=2Gi \
  --helm-set service.enabled=true \
  --helm-set service.type=LoadBalancer \
  --helm-set service.port=5678 \
  --helm-set postgresql.enabled=true \
  --helm-set postgresql.auth.database=n8n \
  --helm-set postgresql.auth.username=n8n \
  --helm-set postgresql.auth.password=kaLIst01! \
  --helm-set postgresql.primary.persistence.size
