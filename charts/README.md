# Charts Helm para Kubernetes

Este repositório contém um conjunto de charts Helm para implantação no Kubernetes.

## Estrutura

- **boilerplate/**: Chart base que funciona como biblioteca para os outros charts
- **postgres-example/**: Exemplo de chart para PostgreSQL usando o boilerplate

## Conceito

A ideia central deste repositório é proporcionar um boilerplate flexível (`boilerplate/`) que possa ser usado como dependência em diferentes charts específicos (como PostgreSQL, Redis, aplicações, etc).

### Boilerplate como biblioteca

O chart `boilerplate/` é configurado como um "library chart" (`type: library` no Chart.yaml), o que significa que ele fornece templates reutilizáveis para outros charts, mas não cria recursos por si só.

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
    version: ">=0.1.0"
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
```

4. Adicione templates específicos em `templates/` conforme necessário.

## Exemplos

Consulte o diretório `postgres-example/` para um exemplo completo de como utilizar o boilerplate. 