# PostgreSQL usando o Boilerplate

Este é um exemplo de como usar o chart boilerplate como dependência para criar um serviço PostgreSQL no Kubernetes.

## Características

- Deployment do PostgreSQL 13.4 (nomeado como `release-name-deployment`)
- Serviço para acessar o PostgreSQL (nomeado como `release-name-service`)
- ConfigMap com configurações personalizadas (nomeado como `release-name-configmap`)
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

- Credenciais (usuário, senha, nome do banco)
- Configurações de recursos (CPU, memória)
- Configurações de persistência (tamanho, modo de acesso)

Você pode sobrescrever qualquer valor no momento da instalação:

```bash
helm install meu-postgres ./charts/postgres-example --set helm-boilerplate.deployments.postgres.env[0].value=admin
```

## Como funciona

Este chart demonstra como utilizar o boilerplate como um conjunto de templates reutilizáveis. O chart faz:

1. Referência ao boilerplate como dependência no `Chart.yaml`
2. Usa a estrutura de valores do boilerplate sob a chave `helm-boilerplate` no `values.yaml`
3. Todos os recursos são criados através do boilerplate, com nomes padronizados:
   - `release-name-deployment`
   - `release-name-service`
   - `release-name-configmap`
   - `release-name-pvc`

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
  deployments:
    postgres:
      replicas: 3
      image:
        tag: "14.0"
      env:
      - name: POSTGRES_USER
        value: admin
      - name: POSTGRES_PASSWORD
        value: senha-segura
      - name: POSTGRES_DB
        value: minhaapp

  persistentVolumeClaims:
    postgres-data:
      storage: 100Gi
      storageClassName: premium
```

E aplique-o na instalação:

```bash
helm install meu-postgres ./charts/postgres-example -f valores-customizados.yaml
``` 