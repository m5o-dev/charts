# GitHub Actions Runner Controller

Este chart instala o [Actions Runner Controller (ARC)](https://github.com/actions/actions-runner-controller), uma solução para rodar GitHub Actions no Kubernetes.

## Pré-requisitos

- Kubernetes 1.16+
- Helm 3.0+
- Credenciais do GitHub (GitHub App ou Personal Access Token)

## Instalação

### 1. Configuração das credenciais do GitHub

Existem duas formas de autenticação com o GitHub:

#### Opção 1: GitHub App (Recomendado)

1. [Crie um GitHub App](https://docs.github.com/en/developers/apps/creating-a-github-app) com as seguintes permissões:
   - Repository permissions:
     - Actions: Read-only
     - Administration: Read-only
     - Metadata: Read-only
   - Organization permissions (para runners de organização):
     - Self-hosted runners: Read & write

2. Anote os valores:
   - GitHub App ID
   - GitHub App Installation ID
   - GitHub App Private Key (arquivo .pem)

3. Edite o arquivo `values.yaml` e atualize:
   ```yaml
   authSecret:
     github_app_id: "seu-app-id"
     github_app_installation_id: "seu-installation-id"
     github_app_private_key: |
       -----BEGIN RSA PRIVATE KEY-----
       ...
       -----END RSA PRIVATE KEY-----
   ```

#### Opção 2: Personal Access Token (PAT)

1. [Crie um PAT](https://github.com/settings/tokens) com os escopos:
   - `repo` (para repositórios privados)
   - `admin:org` (para runners da organização)

2. Edite o arquivo `values.yaml` e atualize:
   ```yaml
   authSecret:
     github_token: "seu-token"
   ```

### 2. Instalar o controller

```bash
helm dependency update ./charts/runner-controller
helm install runner-controller ./charts/runner-controller -n github-actions --create-namespace
```

## Configuração de runners

Após a instalação do controller, você pode implantar runners usando o chart `runners-github`:

```bash
helm install runners-github ./charts/runners-github -n github-actions
```

### Configurações dos runners

Edite o arquivo `charts/runners-github/values.yaml` para configurar os runners:

- Para um repositório específico:
  ```yaml
  runners:
    repo-runner:
      enabled: true
      type: RunnerDeployment
      replicas: 2
      repository: "seu-usuario/seu-repositorio"
      labels:
        - kubernetes
  ```

- Para uma organização:
  ```yaml
  runners:
    org-runner:
      enabled: true
      type: RunnerDeployment
      replicas: 2
      organization: "sua-organizacao"
      labels:
        - kubernetes
  ```

- Para auto-scaling avançado:
  ```yaml
  runners:
    autoscaling-runner:
      enabled: true
      type: RunnerSet
      organization: "sua-organizacao"
      labels:
        - kubernetes
      autoscaling:
        minReplicas: 1
        maxReplicas: 5
        metrics:
          # Escala baseada em porcentagem de ocupação
          - type: PercentageRunnersBusy
            scaleUpThreshold: '0.75'
            scaleDownThreshold: '0.25'
            scaleUpFactor: '2'
            scaleDownFactor: '0.5'
          # Escala baseada em jobs na fila
          - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
            scaleUpThreshold: '1'
            scaleDownThreshold: '0'
            scaleUpAdjustment: 1
            scaleDownAdjustment: 1
  ```

## Uso nos workflows do GitHub Actions

Depois de configurar os runners, você pode utilizá-los em seus workflows do GitHub Actions especificando a label do runner:

```yaml
jobs:
  build:
    runs-on: kubernetes
    steps:
      - uses: actions/checkout@v3
      # outros passos aqui
```

## Manutenção

### Limpeza de pods completados

Os runners são configurados para limpeza automática após conclusão dos jobs. Para limpeza manual:

```bash
kubectl delete pods --field-selector=status.phase==Succeeded -n github-actions
```

## Documentação adicional

Para configurações avançadas, consulte a [documentação oficial do ARC](https://github.com/actions/actions-runner-controller). 