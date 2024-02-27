# demo-features-kind-lab

Lab deploying various infrastructure to a kind cluster

## Cluster With Tyk and Monitoring

```bash
# create cluster
# with kind
kind create cluster --config labs/00-kind/kind.yml

# setup monitoring
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# ns
kubectl apply -f labs/01-monitoring/prep/ns.yaml

# rbac
kubectl -n monitoring apply -f labs/01-monitoring/rbac

# promtail
helm upgrade -i promtail grafana/promtail \
    --version 6.15.3 \
    --namespace monitoring \
    --create-namespace \
    -f labs/01-monitoring/promtail.yml

# loki
helm upgrade -i loki grafana/loki \
    --version 5.42.0 \
    --namespace monitoring \
    --create-namespace \
    -f labs/01-monitoring/loki.yml

# prom-stack
helm upgrade -i prometheus prometheus-community/kube-prometheus-stack \
    --version 56.1.0 \
    --namespace monitoring \
    --create-namespace \
    -f labs/01-monitoring/prometheus.yml

# setup tyk
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add tyk-helm https://helm.tyk.io/public/helm/charts/
helm repo update

# ns
kubectl apply -f labs/02-tyk/prep/ns.yaml

# rbac
kubectl -n tyk apply -f labs/02-tyk/rbac

# redis
helm upgrade -i tyk-redis bitnami/redis \
    --version 18.9.1 \
    --namespace tyk \
    -f labs/02-tyk/redis.yml

# EITHER
# either oss
helm upgrade -i tyk-oss tyk-helm/tyk-oss \
    --version 1.2.0 \
    --namespace tyk \
    -f labs/02-tyk/tyk-oss.yml

# OR
# enterprise

# postgres
helm upgrade -i tyk-postgres bitnami/postgresql \
    --version 13.4.3 \
    --namespace tyk \
    -f labs/02-tyk/tyk-postgres.yml \
    --wait

# tyk stack
helm upgrade -i tyk-stack tyk-helm/tyk-stack \
    --version 1.0.0 \
    --namespace tyk \
    -f labs/02-tyk/tyk-stack.yml

# nodeport svc
kubectl -n tyk apply -f labs/02-tyk/nodeport-svc

# info
echo "Access Grafana at http://localhost:30080 with user 'admin' and password 'prom-operator'"
echo "Access Tyk at http://localhost:30182 with user 'default@example.com' and password '123456'"

# cleanup
cleanup the cluster with `kind delete cluster --name demo`
```

## Monitoring With Tyk and Monitoring using Umbrella Helm Chart

```bash
# create cluster
# with kind
kind create cluster --config labs/00-kind/kind.yml

# ns
kubectl apply -f labs/03-helm/prep

# redis
helm upgrade -i tyk-redis bitnami/redis \
    --version 18.9.1 \
    --namespace tyk \
    -f labs/02-tyk/redis.yml

# promtail
helm upgrade -i promtail charts/promtail \
    --namespace monitoring \
    -f labs/03-helm/promtail.yml

# loki
helm upgrade -i loki grafana/loki \
    --version 5.42.0 \
    --namespace monitoring \
    -f labs/01-monitoring/loki.yml

# prom-stack
helm upgrade -i prometheus prometheus-community/kube-prometheus-stack \
    --version 56.1.0 \
    --namespace monitoring \
    -f labs/01-monitoring/prometheus.yml

# tyk
helm upgrade -i tyk-oss tyk-helm/tyk-oss \
    --version 1.2.0 \
    --namespace tyk \
    -f labs/02-tyk/tyk-oss.yml

# info
echo "Access Grafana at http://localhost:30080 with user 'admin' and password 'prom-operator'"

# cleanup
cleanup the cluster with `kind delete cluster --name demo`
```
