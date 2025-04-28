# Helm Boilerplate

Este é um chart Helm boilerplate que serve como biblioteca para criar recursos Kubernetes. O chart foi projetado como uma biblioteca (`type: library`) para ser utilizado como dependência em outros charts Helm.

## Funcionalidades

- Criação dinâmica de Deployments com base nas configurações do values.yaml
- Criação dinâmica de Services com base nas configurações do values.yaml
- Criação dinâmica de ConfigMaps com base nas configurações do values.yaml
- Criação dinâmica de Secrets com base nas configurações do values.yaml
- Criação dinâmica de PersistentVolumeClaims com base nas configurações do values.yaml
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
    ├── secrets.yaml      # Template para Secrets
    ├── pvc.yaml          # Template para PersistentVolumeClaims
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

3. Construa as dependências do chart antes de instalar ou empacotar:

```bash
# Constrói as dependências definidas no Chart.yaml
helm dependency build ./seu-chart

# Ou atualize se as dependências já foram construídas anteriormente
helm dependency update ./seu-chart
```

4. Adicione templates específicos para recursos não suportados pelo boilerplate.

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
- Sempre execute `helm dependency build` ou `helm dependency update` antes de empacotar ou instalar charts que têm dependências
- Os nomes dos recursos são usados exatamente como definidos no values.yaml, sem sufixos adicionais

## Configuração

O arquivo `values.yaml` é estruturado para permitir a definição de múltiplos recursos:

### Namespace (opcional)

```yaml
# Configuração de namespace (opcional)
namespace: default
```

### Deployments

```yaml
deployments:
  app:
    enabled: true                    # Habilita a criação do deployment
    replicas: 2                      # Número de réplicas
    image:
      repository: nginx              # Repositório da imagem
      tag: "1.21.0"                  # Tag da imagem
      pullPolicy: IfNotPresent       # Política de pull (IfNotPresent, Always, Never)
    command:                         # Comando a ser executado (opcional)
      - "/bin/sh"
      - "-c"
    args:                            # Argumentos para o comando (opcional)
      - "nginx -g 'daemon off;'"
    ports:                           # Portas expostas pelo container
      - name: http                   # Nome da porta
        containerPort: 80            # Porta do container
        protocol: TCP                # Protocolo (TCP ou UDP)
    env:                             # Variáveis de ambiente
      - name: NGINX_HOST             # Nome da variável
        value: example.com           # Valor direto
      - name: POD_NAME               # Variável com valor do Kubernetes
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: DB_HOST                # Variável de ConfigMap
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: db_host
      - name: DB_PASSWORD            # Variável de Secret
        valueFrom:
          secretKeyRef:
            name: app-secrets
            key: db_password
    resources:                       # Recursos de CPU e memória
      limits:                        # Limites máximos
        cpu: 500m                    # 500 milicores = 0.5 CPU
        memory: 512Mi                # 512 Mebibytes
      requests:                      # Requisições mínimas
        cpu: 100m                    # 100 milicores = 0.1 CPU
        memory: 128Mi                # 128 Mebibytes
    volumeMounts:                    # Montagem de volumes no container
      - name: config-volume
        mountPath: /etc/config
        subPath: myconfig            # Subdiretório dentro do volume (opcional)
    volumes:                         # Volumes para o pod
      - name: config-volume
        configMap:
          name: app-config
      - name: data-volume
        persistentVolumeClaim:
          claimName: app-data
      - name: secret-volume
        secret:
          secretName: app-secrets
```

### Services

```yaml
services:
  app:
    enabled: true                    # Habilita a criação do serviço
    type: ClusterIP                  # Tipo do serviço (ClusterIP, NodePort, LoadBalancer)
    ports:                           # Portas do serviço
      - name: http                   # Nome da porta
        port: 80                     # Porta do serviço
        targetPort: http             # Porta alvo (número ou nome)
        nodePort: 30080              # Porta do nó (se NodePort, opcional)
        protocol: TCP                # Protocolo (TCP ou UDP)
    selector:                        # Seletor personalizado para os pods (opcional)
      app: app                       # Se não definido, usa app=$name e release=$Release.Name
```

### ConfigMaps

```yaml
configMaps:
  app-config:
    enabled: true                    # Habilita a criação do ConfigMap
    data:                            # Dados do ConfigMap
      config.yaml: |
        server:
          port: 8080
          host: 0.0.0.0
      db_host: db.example.com
      allowed_origins: |
        https://example.com
        https://api.example.com
```

### Secrets

```yaml
secrets:
  app-secrets:
    enabled: true                    # Habilita a criação do Secret
    type: Opaque                     # Tipo do Secret (Opaque, kubernetes.io/tls, etc.)
    data:                            # Dados do Secret (serão codificados em base64 automaticamente)
      db_password: senha123
      api_key: minha-chave-secreta
    stringData:                      # Dados do Secret que não precisam ser codificados em base64
      config.json: |
        {
          "clientId": "client123",
          "environment": "production"
        }
  tls-cert:
    enabled: true
    type: kubernetes.io/tls
    data:
      tls.crt: cert-content-here
      tls.key: key-content-here
```

### PersistentVolumeClaims

```yaml
persistentVolumeClaims:
  app-data:
    enabled: true                    # Habilita a criação do PVC
    name: app                        # Nome da aplicação para labels (opcional)
    accessModes:                     # Modos de acesso
      - ReadWriteOnce                # Outras opções: ReadOnlyMany, ReadWriteMany
    storageClassName: local-path     # Classe de armazenamento
    storage: 10Gi                    # Tamanho do volume
    annotations:                     # Anotações (opcional)
      helm.sh/resource-policy: keep  # Mantém o PVC quando o release é removido
    selector:                        # Seletor para vincular a PVs específicos (opcional)
      matchLabels:
        type: ssd
```

### IngressRoutes (Traefik)

```yaml
ingressRoutes:
  app:
    enabled: true                    # Habilita a criação do IngressRoute
    entryPoints:                     # Pontos de entrada do Traefik
      - web                          # Nome do ponto de entrada
    routes:                          # Rotas do IngressRoute
      - match: Host(`example.com`)   # Regra de match para a rota
        kind: Rule                   # Tipo da regra
        services:                    # Serviços para a rota
          - name: app                # Nome do serviço
            port: 80                 # Porta do serviço
```

### Rollouts (Argo)

```yaml
rollouts:
  app:
    enabled: true                    # Habilita a criação do Rollout
    replicas: 2                      # Número de réplicas
    image:
      repository: nginx              # Repositório da imagem
      tag: "1.21.0"                  # Tag da imagem
      pullPolicy: IfNotPresent       # Política de pull
    strategy:
      type: RollingUpdate            # Tipo de estratégia
      rollingUpdate:
        maxSurge: 1                  # Número máximo de pods acima do desired
        maxUnavailable: 0            # Número máximo de pods indisponíveis
    template:
      containers:
        - name: app                  # Nome do container
          image: nginx:1.21.0        # Imagem do container
          ports:
            - name: http             # Nome da porta
              containerPort: 80      # Porta do container
              protocol: TCP          # Protocolo
```

## Exemplos

Veja o diretório `charts/postgres-example` para um exemplo completo de uso do boilerplate. 