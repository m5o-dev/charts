# Helm Boilerplate

Este é um chart Helm boilerplate que serve como base para projetos Kubernetes, permitindo a criação flexível de recursos como Deployments, Services, ConfigMaps, IngressRoutes e Rollouts através de arquivos de valores.

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
chart/
├── Chart.yaml            # Metadados do chart
├── values.yaml           # Valores padrão de configuração
└── templates/            # Templates dos recursos Kubernetes
    ├── deployment.yaml   # Template para Deployments
    ├── service.yaml      # Template para Services
    ├── configmap.yaml    # Template para ConfigMaps
    ├── ingressroute.yaml # Template para IngressRoutes do Traefik
    └── rollout.yaml      # Template para Argo Rollouts
```

## Uso

1. Ajuste o arquivo `values.yaml` de acordo com suas necessidades
2. Instale o chart:

```bash
helm install meu-app ./chart
```

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
    enabled: false                     # Habilita a criação do rollout (desabilitado por padrão)
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

## Personalização

Para personalizar o chart, você pode:

1. Adicionar novos tipos de recursos no arquivo `values.yaml`
2. Adicionar novos templates na pasta `templates/`
3. Estender as definições de recursos existentes no values.yaml 