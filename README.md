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