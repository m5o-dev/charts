# Redis usando o Boilerplate

Este é um exemplo de como usar o chart boilerplate como dependência para criar um serviço Redis no Kubernetes.

## Características

- Deployment do Redis 8-alpine (versão leve)
- Serviço para acessar o Redis
- ConfigMap com configurações personalizadas
- Secret para armazenar credenciais de forma segura
- PersistentVolumeClaim para armazenamento dos dados

## Instalação

Para instalar o chart:

```bash
# Primeiro, atualize as dependências
helm dependency update ./charts/redis

# Depois instale o chart
helm install meu-redis ./charts/redis
```

## Configuração

O arquivo `values.yaml` contém todas as configurações padrão:

- Senha definida diretamente no Secret
- Configurações de recursos (CPU, memória)
- Configurações de persistência (tamanho, modo de acesso)
- Configurações básicas do Redis (maxmemory, policies, etc.)

Você pode sobrescrever qualquer valor no momento da instalação:

```bash
# Sobrescrevendo a senha no Secret
helm install meu-redis ./charts/redis --set helm-boilerplate.secrets.redis-credentials.data.REDIS_PASSWORD=senhaSegura123
```

## Segurança de Credenciais

As credenciais do Redis são armazenadas em um Secret do Kubernetes, oferecendo maior segurança:

- A senha é definida no Secret com REDIS_PASSWORD
- Injetada automaticamente no container via `envFrom.secretRef`
- Utilizada nos argumentos de inicialização do Redis

## Como funciona

Este chart demonstra como utilizar o boilerplate como um conjunto de templates reutilizáveis. O chart faz:

1. Referência ao boilerplate como dependência no `Chart.yaml`
2. Usa a estrutura de valores do boilerplate sob a chave `helm-boilerplate` no `values.yaml`
3. Define o Secret com a senha usando o nome de variável REDIS_PASSWORD
4. Configura o Deployment para carregar a variável do Secret automaticamente
5. Todos os recursos são criados através do boilerplate, com nomes padronizados

## Estrutura

```
redis/
├── Chart.yaml           # Referencia o boilerplate como dependência
└── values.yaml          # Configurações customizadas para o Redis
```

## Valores customizados

Para personalizar, crie um arquivo de valores personalizado:

```yaml
# valores-customizados.yaml
helm-boilerplate:
  # Secret com senha personalizada
  secrets:
    redis-credentials:
      data:
        REDIS_PASSWORD: senha-segura-personalizada

  deployments:
    redis:
      replicas: 3
      image:
        tag: "latest"
      resources:
        limits:
          memory: 2Gi

  persistentVolumeClaims:
    redis-data:
      storage: 10Gi
      storageClassName: premium
```

E aplique-o na instalação:

```bash
helm install meu-redis ./charts/redis -f valores-customizados.yaml
```

## Conectando ao Redis

Para conectar a um cliente Redis dentro do cluster, use o seguinte serviço:

```
meu-redis-redis:6379
```

A senha é a mesma definida no Secret como REDIS_PASSWORD. 