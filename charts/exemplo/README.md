# Chart Helm: Exemplo

Este é um chart de exemplo para demonstrar como funciona a estrutura de charts Helm neste repositório.

## Instalação

```bash
helm repo add m5o-charts https://m5o.dev/charts
helm install meu-exemplo m5o-charts/exemplo
```

## Parâmetros

| Parâmetro | Descrição | Valor Padrão |
|-----------|-----------|--------------|
| `replicaCount` | Número de réplicas para o deployment | `1` |
| `image.repository` | Repositório da imagem do container | `nginx` |
| `image.pullPolicy` | Política de pull da imagem | `IfNotPresent` |
| `image.tag` | Tag específica da imagem (sobrepõe o appVersion) | `""` |
| `service.type` | Tipo do serviço Kubernetes | `ClusterIP` |
| `service.port` | Porta exposta pelo serviço | `80` |
| `resources.limits.cpu` | Limite de CPU | `100m` |
| `resources.limits.memory` | Limite de memória | `128Mi` |
| `resources.requests.cpu` | Request de CPU | `100m` |
| `resources.requests.memory` | Request de memória | `128Mi` |
| `autoscaling.enabled` | Habilitar autoscaling | `false` |
| `autoscaling.minReplicas` | Mínimo de réplicas para autoscaling | `1` |
| `autoscaling.maxReplicas` | Máximo de réplicas para autoscaling | `3` |
| `autoscaling.targetCPUUtilizationPercentage` | Alvo de utilização de CPU para escalar | `80` | 