# k8s-gitops

Manifestos Kubernetes dos serviços `KeyssonG`, atualizados automaticamente pelo Jenkins via GitOps.

Este repositório é **somente leitura para humanos**: ele não deve ser alterado manualmente em produção
(apenas via PR/review). O Jenkins atualiza a imagem dos `Deployment` após cada build/push no Docker Hub.

## Serviços

| Pasta                | Serviço                  | Manifesto de Deployment                       |
|----------------------|--------------------------|-----------------------------------------------|
| `nexus/`             | Backend `nexus-multithread` | `nexus/nexus-deployment.yaml`              |
| `front-multithread/` | Frontend `api-multithread`  | `front-multithread/multithread.deployment.yaml` |
| `msgsend/`           | Backend `msgsend-service`   | `msgsend/msgsend-deployment.yaml`          |
| `front-multithread-interno/` | Portal `api-portal-multithread` | `front-multithread-interno/multithread-interno.deployment.yaml` |

## Fluxo (GitOps com Jenkins)

```
1. Commit de código no repositório da aplicação (ex.: nexus-multithread)
   └─ pollSCM (5 min) detecta e dispara o Jenkins
2. Jenkins: docker build → docker push (Docker Hub)
3. Jenkins: clona este repo (k8s-gitops), atualiza a imagem do Deployment
   └─ commit + push APENAS neste repositório
4. Aplicar no cluster: kubectl apply -f k8s-gitops/<servico>/...
```

O repositório da aplicação **nunca** recebe commits do Jenkins — o histórico fica limpo e o
`master` local nunca fica atrás do remoto por causa de CI.

## Aplicar no cluster

```bash
# Namespace (primeira vez)
kubectl apply -f nexus/nexus-namespace.yaml

# Backend
kubectl apply -f nexus/nexus-deployment.yaml
kubectl apply -f nexus/nexus-service.yaml

# Frontend
kubectl apply -f front-multithread/multithread.deployment.yaml
kubectl apply -f front-multithread/multithread-service.yaml

# msgsend
kubectl apply -f msgsend/msgsend-deployment.yaml
kubectl apply -f msgsend/msgsend-service.yaml

# Portal interno
kubectl apply -f front-multithread-interno/multithread-interno.deployment.yaml
kubectl apply -f front-multithread-interno/multithread-interno-service.yaml
```

## Credenciais

Nenhuma credencial está versionada neste repositório. O Jenkins usa o credential
`GitHub` (variáveis `GIT_USER`/`GIT_TOKEN`) apenas no momento do push, injetadas em runtime.
