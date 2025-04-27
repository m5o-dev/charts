# PostgreSQL usando o Boilerplate

Este é um exemplo de como usar o chart boilerplate como dependência para criar um serviço PostgreSQL no Kubernetes.

## Características

- Deployment do PostgreSQL 13.4 (nomeado como `release-name-deployment`)
- Serviço para acessar o PostgreSQL (nomeado como `release-name-service`)
- ConfigMap com configurações personalizadas (nomeado como `release-name-configmap`)
- Secret para armazenar credenciais de forma segura (nomeado como `postgres-credentials`)
- PersistentVolumeClaim para armazenamento dos dados (nomeado como `release-name-pvc`)

## Instalação

Para instalar o chart:

```bash
# Primeiro, atualize as dependências
helm dependency update ./charts/postgres-example

# Depois instale o chart
helm install meu-postgres ./charts/postgres-example
```

## Configuração

O arquivo `values.yaml` contém todas as configurações padrão:

- Credenciais definidas diretamente no Secret (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB)
- Configurações de recursos (CPU, memória)
- Configurações de persistência (tamanho, modo de acesso)

Você pode sobrescrever qualquer valor no momento da instalação:

```bash
# Sobrescrevendo uma credencial no Secret
helm install meu-postgres ./charts/postgres-example --set helm-boilerplate.secrets.postgres-credentials.data.POSTGRES_PASSWORD=senhaSegura123
```

## Segurança de Credenciais

As credenciais do PostgreSQL são armazenadas em um Secret do Kubernetes, oferecendo maior segurança:

- Definidas diretamente no Secret com POSTGRES_USER, POSTGRES_PASSWORD e POSTGRES_DB
- Injetadas automaticamente no container via `envFrom.secretRef`
- Não é necessário referenciar manualmente cada variável no deployment
- Implementação simplificada e menos propensa a erros

## Como funciona

Este chart demonstra como utilizar o boilerplate como um conjunto de templates reutilizáveis. O chart faz:

1. Referência ao boilerplate como dependência no `Chart.yaml`
2. Usa a estrutura de valores do boilerplate sob a chave `helm-boilerplate` no `values.yaml`
3. Define o Secret com as credenciais usando os nomes de variáveis esperados pelo PostgreSQL
4. Configura o Deployment para carregar todas as variáveis do Secret automaticamente usando `envFrom`
5. Todos os recursos são criados através do boilerplate, com nomes padronizados

## Estrutura

```
postgres-example/
├── Chart.yaml           # Referencia o boilerplate como dependência
└── values.yaml          # Configurações customizadas para o PostgreSQL
```

## Valores customizados

Para personalizar, crie um arquivo de valores personalizado:

```yaml
# valores-customizados.yaml
helm-boilerplate:
  # Secret com credenciais personalizadas
  secrets:
    postgres-credentials:
      data:
        POSTGRES_USER: admin
        POSTGRES_PASSWORD: senha-segura-personalizada
        POSTGRES_DB: minhaapp

  deployments:
    postgres:
      replicas: 3
      image:
        tag: "14.0"
      resources:
        limits:
          memory: 4Gi

  persistentVolumeClaims:
    postgres-data:
      storage: 100Gi
      storageClassName: premium
```

E aplique-o na instalação:

```bash
helm install meu-postgres ./charts/postgres-example -f valores-customizados.yaml
``` 