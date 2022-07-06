# API Gateway com Kong e Kubernetes

### Principais Conceitos

- Rotas
- Services
- Plugins
- Consumers
- Upstreams
- Targets

| Downstream | Proxy   | Upstream  |
| ---------- | ------- | --------- |
| Consumers  | Rotas   | Services  |
| -          | Plugins | Upstreams |
| -          | -       | Targets   |

### Kong e Kubernetes

**Kubernetes Ingress**
É a maneira de realizar a exposição de rotas HTTP e HTTPS para fora do cluster.
Este roteamento de tráfego é controlado por regras definidas dentro do recurso **Ingress** do Kubernetes.

**Tradução K*s -> Kong**
| Ingress rules | Kubernetes Service | Pods    |
| ------------- | ------------------ | ------- |
| Routes        | Service            | Targets |
| -             | Upstream           | -       |
| -             | -                  | -       |

### Deployments

**Usando banco de dados**
![](./.github/control-and-data-plane-kong.png)

**DB-Less**
![](./.github/controller-and-data-plane-db-less.png)

### Ferramentas necessárias
- Kind || minikube || microk8s
- kubectl
- Helm v3


**Criando clusters no Kind**
```bash
./infra/kong-k8s/kind/kind.sh
```

**Instalando Kong**
```bash
./infra/kong-k8s/kong/kong.sh

> helm repo add kong https://charts.konghq.com

> helm repo update

> kubectl create ns kong

> helm install kong/kong --generate-name -f kong-conf.yaml --set proxy.type=NodePort,proxy.http.nodePort=30000,proxy.tls.nodePort=30003 --set ingressController.installCRDs=false --set serviceMonitor.enabled=true --set serviceMonitor.labels.release=promstack --namespace kong
```

**Instalando Prometheus**
```bash
./infra/kong-k8s/misc/prometheus/prometheus.sh

> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

> helm repo update

> kubectl create ns monitoring

> helm install prometheus-stack prometheus-community/kube-prometheus-stack -f prometheus.yaml --namespace monitoring
```

**Instalando Keycloak**
```bash
./infra/kong-k8s/misc/keycloak/keycloak.sh

> helm repo add bitnami https://charts.bitnami.com/bitnami

> helm repo update

> kubectl create ns iam

> helm install keycloak bitnami/keycloak --set auth.adminUser=keycloak,auth.adminPassword=keycloak --namespace iam
```

**Instalando ConfigMaps, Deployments e HPA**
```bash
./infra/kong-k8s/misc/apps/
kubectl apply -f ./infra/kong-k8s/misc/apps/ --recursive -n bets
```

### Kong Custom Resource Definitions

**Rate Limit**
```bash
kubectl apply -f ./infra/kong-k8s/misc/apis/kratelimit.yaml -n bets
```

**Plugin Prometheus**
```bash
kubectl apply -f ./infra/kong-k8s/misc/apis/kprometheus.yaml
```

**Ingress**
```bash
kubectl apply -f ./infra/kong-k8s/misc/apis/bets-api.yaml -n bets
kubectl apply -f ./infra/kong-k8s/misc/apis/king.yaml -n bets
```

### Ver requisições no Kong
```bash
> kubectl get pods -n kong
NAME                                    READY   STATUS    RESTARTS     AGE
kong-1657034189-kong-5f7bf46587-87lvb   2/2     Running   1 (9h ago)   9h

> kubectl describe pod kong-1657034189-kong-5f7bf46587-87lvb -n kong
Containers:
  ingress-controller:
  .
  .
  proxy:


> kubectl logs kong-1657034189-kong-5f7bf46587-87lvb proxy -f -n kong
10.244.0.1 - - [06/Jul/2022:00:36:52 +0000] "POST /api/bets HTTP/1.1" 201 134 "-" "PostmanRuntime/7.29.0"
10.244.0.1 - - [06/Jul/2022:00:36:53 +0000] "POST /api/bets HTTP/1.1" 201 134 "-" "PostmanRuntime/7.29.0"
10.244.0.1 - - [06/Jul/2022:00:36:54 +0000] "POST /api/bets HTTP/1.1" 201 134 "-" "PostmanRuntime/7.29.0"

```