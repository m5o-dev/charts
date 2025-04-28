# Charts Helm para Kubernetes

Este repositório contém um conjunto de charts Helm para implantação no Kubernetes.

## Estrutura

- **boilerplate/**: Chart base que funciona como biblioteca para os outros charts
- **postgresql/**: Chart para PostgreSQL usando o boilerplate

## Conceito

A ideia central deste repositório é proporcionar um boilerplate flexível (`boilerplate/`) que possa ser usado como dependência em diferentes charts específicos (como PostgreSQL, Redis, aplicações, etc).

### Boilerplate como biblioteca

O chart `boilerplate/` é configurado como um "application chart" (`type: application` no Chart.yaml), o que significa que ele pode ser usado diretamente ou como dependência em outros charts.

Benefícios desta abordagem:
1. Código compartilhado e centralizado
2. Manutenção simplificada - atualize o boilerplate uma vez, todos os charts se beneficiam
3. Consistência entre diferentes serviços

## Como criar um novo chart

Para criar um novo chart que utilize o boilerplate:

1. Crie um novo diretório para seu chart:
```bash
mkdir -p charts/meu-servico
```

2. Crie um Chart.yaml com a dependência para o boilerplate:
```yaml
apiVersion: v2
name: meu-servico
description: Meu serviço específico
type: application
version: 0.1.0
appVersion: "1.0.0"

dependencies:
  - name: helm-boilerplate
    version: ">=0.0.2"
    repository: "file://../boilerplate"
```

3. Crie um values.yaml configurando os recursos através do boilerplate:
```yaml
helm-boilerplate:
  deployments:
    meu-app:
      enabled: true
      # Configurações do deployment
  
  services:
    meu-app:
      enabled: true
      # Configurações do serviço
      
  secrets:
    meu-secrets:
      enabled: true
      type: Opaque
      data:
        # Dados do secret
```

4. Adicione templates específicos em `templates/` conforme necessário.

## Exemplos

### Chart PostgreSQL

O chart `postgresql/` demonstra como utilizar o boilerplate para criar uma instância de PostgreSQL no Kubernetes.

#### Características

- Deployment do PostgreSQL 13.4
- Serviço para acessar o PostgreSQL
- ConfigMap com configurações personalizadas
- Secret para armazenar credenciais de forma segura
- PersistentVolumeClaim para armazenamento dos dados

#### Segurança de Credenciais

O chart utiliza uma abordagem simplificada para gerenciar credenciais:

- As credenciais são definidas diretamente no Secret com os nomes esperados pelo PostgreSQL (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB)
- O Deployment usa `envFrom` para injetar automaticamente todas as variáveis do Secret
- Não é necessário referenciar manualmente cada variável no deployment

#### Instalação

```bash
# Primeiro, atualize as dependências
helm dependency update ./charts/postgresql

# Depois instale o chart
helm install postgres-db ./charts/postgresql -n postgresql
```

#### Configuração

O arquivo `values.yaml` contém todas as configurações padrão:

- Credenciais definidas diretamente no Secret
- Configurações de recursos (CPU, memória)
- Configurações de persistência (tamanho, modo de acesso)

Você pode sobrescrever qualquer valor no momento da instalação:

```bash
# Sobrescrevendo uma credencial no Secret
helm install postgres-db ./charts/postgresql -n postgresql --set helm-boilerplate.secrets.postgres-credentials.data.POSTGRES_PASSWORD=senhaSegura123
```

#### Valores customizados

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
helm install postgres-db ./charts/postgresql -n postgresql -f valores-customizados.yaml
``` 