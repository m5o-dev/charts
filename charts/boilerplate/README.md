# Helm Boilerplate

Este é um chart Helm boilerplate que serve como biblioteca para criar recursos Kubernetes. O chart foi projetado como uma biblioteca (`type: library`) para ser utilizado como dependência em outros charts Helm.

## Funcionalidades

- Criação dinâmica de Deployments com base nas configurações do values.yaml
- Criação dinâmica de Services com base nas configurações do values.yaml
- Criação dinâmica de ConfigMaps com base nas configurações do values.yaml
- Criação dinâmica de IngressRoutes do Traefik com base nas configurações do values.yaml
- Criação dinâmica de Argo Rollouts com base nas configurações do values.yaml
- Configuração flexível através de values customizáveis
- Nomeação de recursos baseada no nome do Release
- Sem uso de _helpers.tpl para manter o código mais limpo e direto

## Estrutura

```
boilerplate/
├── Chart.yaml            # Metadados do chart (type: library)
├── values.yaml           # Valores padrão de configuração
└── templates/            # Templates dos recursos Kubernetes
    ├── deployment.yaml   # Template para Deployments
    ├── service.yaml      # Template para Services
    ├── configmap.yaml    # Template para ConfigMaps
    ├── ingressroute.yaml # Template para IngressRoutes do Traefik
    └── rollout.yaml      # Template para Argo Rollouts
```

## Dois modos de uso

### 1. Como dependência em outros charts

O método recomendado é usar este chart como dependência em seus charts específicos (PostgreSQL, Redis, aplicações, etc):

1. Adicione o boilerplate como dependência no seu `Chart.yaml`:

```yaml
dependencies:
  - name: helm-boilerplate
    version: ">=0.1.0"
    repository: "file://../boilerplate"  # Ou use um repositório Helm real
```

2. Configure os recursos no `values.yaml` do seu chart, sob a chave `helm-boilerplate`:

```yaml
helm-boilerplate:
  deployments:
    myapp:
      enabled: true
      # Outras configurações...
```

3. Adicione templates específicos para recursos não suportados pelo boilerplate.

Veja o diretório `charts/postgres-example` para um exemplo completo.

### 2. Como chart independente com valores customizados

Para casos simples, você também pode usar o boilerplate diretamente:

1. Crie um arquivo `values-custom.yaml` com os recursos que deseja habilitar:

```yaml
deployments:
  myapp:
    enabled: true  # Importante: habilita a criação do recurso
    # Outras configurações do deployment
```

2. Instale o chart usando seu arquivo de valores personalizado:

```bash
helm install meu-app ./charts/boilerplate -f values-custom.yaml
```

## Importante

- Por padrão, **nenhum recurso será criado** ao instalar o chart sem valores personalizados
- Para criar recursos, você deve explicitamente definir `enabled: true` para cada recurso em seu arquivo de valores
- Os exemplos no arquivo values.yaml principal estão todos com `enabled: false`

## Configuração

O arquivo `values.yaml` é estruturado para permitir a definição de múltiplos recursos:

### Deployments

```yaml
deployments:
  app1:
    enabled: true         # Habilita a criação do deployment
    replicas: 1           # Número de réplicas
    image:
      repository: nginx   # Repositório da imagem
      tag: "1.21.0"       # Tag da imagem
      pullPolicy: IfNotPresent # Política de pull
    command: []           # Comando para executar no container
    # - "/bin/sh"
    # - "-c"
    args: []              # Argumentos para o comando do container
    # - "nginx -g 'daemon off;'"
    # Outras configurações
```

### Services

```yaml
services:
  app1:
    enabled: true         # Habilita a criação do serviço
    type: ClusterIP       # Tipo do serviço (ClusterIP, NodePort, LoadBalancer)
    ports:                # Portas do serviço
      - name: http
        port: 80
        targetPort: http
        protocol: TCP
    # Seletores de pods
```

### ConfigMaps

```yaml
configMaps:
  app1-config:
    enabled: true         # Habilita a criação do configmap
    data:                 # Dados do configmap
      config.yaml: |
        key1: value1
        key2: value2
      settings.json: |
        {
          "debug": false,
          "logLevel": "info"
        }
```

### IngressRoutes (Traefik)

```yaml
ingressRoutes:
  app1:
    enabled: true                      # Habilita a criação do ingressroute
    entryPoints:                       # Entry points do Traefik
      - web
      - websecure
    routes:                            # Rotas do IngressRoute
      - match: Host(`app1.example.com`) # Regra de match
        kind: Rule                     # Tipo de regra
        priority: 10                   # Prioridade
        services:                      # Serviços de backend
          - name: app1                 # Nome do serviço (será prefixado com Release.Name)
            port: 80                   # Porta do serviço
        middlewares: []                # Middlewares do Traefik
    tls:                               # Configuração TLS
      enabled: false                   # Habilita TLS
      # certResolver: default          # Cert Resolver do Traefik
      # domains:                       # Domínios para o certificado
      #   - main: app1.example.com
      #     sans:
      #       - "*.app1.example.com"
```

### Rollouts (Argo)

```yaml
rollouts:
  app1:
    enabled: true                     # Habilita a criação do rollout
    replicas: 1                        # Número de réplicas
    image:
      repository: nginx                # Repositório da imagem
      tag: "1.21.0"                    # Tag da imagem
    command: []                        # Comando para executar no container
    # - "/bin/sh"
    # - "-c"
    args: []                           # Argumentos para o comando do container
    # - "nginx -g 'daemon off;'"
    strategy:
      type: canary                     # Tipo de estratégia: canary ou blueGreen
      canary:                          # Configuração para estratégia canary
        maxUnavailable: 25%            # Máximo indisponível durante atualização
        maxSurge: 25%                  # Máximo de surge durante atualização
        steps:                         # Passos do canary
          - setWeight: 20              # Define peso de tráfego
          - pause: {duration: 30s}     # Pausa antes do próximo passo
          - setWeight: 40              # Aumenta o peso
      blueGreen:                       # Configuração para estratégia blue-green
        previewService: preview-service # Serviço para preview
        activeService: active-service  # Serviço ativo
        autoPromotionEnabled: false    # Auto-promoção de versão
```

## Criação de novos charts usando o boilerplate

Para criar um novo chart que utilize o boilerplate:

1. Crie uma nova pasta para seu chart: `mkdir -p charts/meu-servico`
2. Adicione um `Chart.yaml` com a dependência para o boilerplate:

```yaml
apiVersion: v2
name: meu-servico
description: Meu serviço usando boilerplate
type: application
version: 0.1.0
appVersion: "1.0.0"

dependencies:
  - name: helm-boilerplate
    version: ">=0.1.0"
    repository: "file://../boilerplate" # Ou seu repositório Helm
```

3. Crie um arquivo `values.yaml` com suas configurações específicas:

```yaml
# Valores para o meu-servico

# Configurações do boilerplate
helm-boilerplate:
  deployments:
    meu-app:
      enabled: true
      # ... configurações específicas
```

4. Adicione templates adicionais específicos em `templates/` conforme necessário.

Veja o exemplo completo em `charts/postgres-example/` para referência. 