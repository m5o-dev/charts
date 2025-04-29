# GitHub Runners

Este chart implementa runners para GitHub Actions no seu cluster Kubernetes, trabalhando em conjunto com o [Actions Runner Controller](https://github.com/actions/actions-runner-controller).

## Pré-requisitos

- Kubernetes 1.16+
- Helm 3.0+
- Actions Runner Controller já instalado no cluster

## Instalação

```bash
helm dependency update ./charts/runners-github
helm install github-runners ./charts/runners-github -n github-actions
```

## Tipos de runners suportados

O chart suporta dois tipos principais de runners:

1. **RunnerDeployment**: Número fixo de runners
2. **RunnerSet**: Runners com auto-scaling baseado na carga de trabalho

## Configuração

### Estrutura do arquivo values.yaml

A configuração dos runners é feita no arquivo `values.yaml`. Cada runner é definido como uma entrada no mapa `runners`:

```yaml
runners:
  nome-do-runner:
    enabled: true
    type: RunnerDeployment | RunnerSet
    replicas: 2          # para RunnerDeployment
    repository: "user/repo"  # para runner específico de repositório
    organization: "org"  # para runner de organização
    labels:
      - label1
      - label2
```

### Exemplos de configuração

#### Runner para repositório específico

```yaml
runners:
  repo-runner:
    enabled: true
    type: RunnerDeployment
    replicas: 2
    repository: "seu-usuario/seu-repositorio"
    labels:
      - kubernetes
      - production
```

#### Runner para organização

```yaml
runners:
  org-runner:
    enabled: true
    type: RunnerDeployment
    replicas: 3
    organization: "sua-organizacao"
    labels:
      - kubernetes
      - staging
```

#### Runner com auto-scaling aprimorado

```yaml
runners:
  autoscaling-runner:
    enabled: true
    type: RunnerSet
    repository: "seu-usuario/seu-repositorio"
    labels:
      - kubernetes
      - ephemeral
    autoscaling:
      minReplicas: 1
      maxReplicas: 5
      metrics:
        # Métrica baseada na porcentagem de runners ocupados
        # - Verifica a cada ~10 segundos a ocupação dos runners ativos
        # - Só considera runners que estão ATIVAMENTE executando jobs (não enfileirados)
        - type: PercentageRunnersBusy
          scaleUpThreshold: '0.75'
          scaleDownThreshold: '0.25'
          scaleUpFactor: '2'
          scaleDownFactor: '0.5'
        # Métrica baseada no número total de jobs enfileirados e em progresso
        # - Responde mais rapidamente a picos de demanda
        # - Escala mesmo quando não há runners suficientes para iniciar os jobs
        - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
          scaleUpThreshold: '1'
          scaleDownThreshold: '0'
          scaleUpAdjustment: 1
          scaleDownAdjustment: 1
```

## Limpeza automática de pods

Os runners são configurados com a anotação `actions.summerwind.dev/cleanup-completed: "true"` para que os pods sejam automaticamente removidos após concluírem os jobs. Isso mantém o cluster limpo e evita acúmulo de recursos inativos.

Para runners efêmeros (ephemeral), cada job é executado em um pod limpo, que é destruído após a conclusão. A flag `RUNNER_FEATURE_FLAG_EPHEMERAL: "true"` é usada para este comportamento.

Para limpar manualmente pods completados:
```bash
kubectl delete pods --field-selector=status.phase==Succeeded -n github-actions
```

## Uso nas GitHub Actions

Uma vez configurados os runners, você pode utilizá-los em seus workflows do GitHub Actions referenciando as labels definidas:

```yaml
name: CI Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: kubernetes # ou qualquer outra label que você configurou
    steps:
      - uses: actions/checkout@v3
      # outros passos
```

## Acessando o Docker daemon do host

O template padrão inclui acesso ao socket do Docker do host. Se você preferir usar outras soluções como Kaniko ou Docker-in-Docker, modifique a configuração de volumes no `values.yaml`.

## Autenticação para registros de container privados

Para adicionar autenticação para registros privados, adicione secrets adicionais e monte-os nos runners.

## Dicas e troubleshooting

- Verifique os logs do controller e dos pods dos runners em caso de problemas
- Use `kubectl get runnerdeployments -n github-actions` para ver o status dos deployments
- Use `kubectl get runners -n github-actions` para ver os runners individuais

## Documentação adicional

Para configurações avançadas, consulte a [documentação oficial do ARC](https://github.com/actions/actions-runner-controller). 