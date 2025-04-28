# ChartMuseum

Este chart instala o ChartMuseum, um servidor de repositório Helm, usando o chart oficial como dependência e o boilerplate para configuração do IngressRoute.

## Características

- Usa o chart oficial do ChartMuseum (v3.10.1)
- Armazenamento local persistente usando Longhorn
- IngressRoute configurado para acesso via HTTP
- Recursos de CPU e memória configurados para performance otimizada
- Suporte para upload e download de charts

## Pré-requisitos

- Kubernetes 1.19+
- Helm 3.0.0+
- PV provisioner com suporte ao modo de acesso ReadWriteOnce
- Traefik como Ingress Controller

## Instalação

1. Primeiro, atualize as dependências do chart:
```bash
helm dependency update ./charts/chartmuseum
```

2. Instale o chart:
```bash
helm install chartmuseum ./charts/chartmuseum -n chartmuseum --create-namespace
```

## Configuração

O arquivo `values.yaml` contém as seguintes seções principais:

### ChartMuseum

```yaml
chartmuseum:
  env:
    open:
      STORAGE: local                    # Backend de armazenamento
      STORAGE_LOCAL_ROOTDIR: /storage   # Diretório de armazenamento
      CHART_URL: http://museum.v3m.ai   # URL base para os charts
      DEBUG: false                      # Modo debug
      DISABLE_API: false                # API habilitada
      ALLOW_OVERWRITE: true            # Permite sobrescrever charts
      AUTH_ANONYMOUS_GET: true         # Permite download anônimo
      DEPTH: 0                         # Profundidade do repositório

  persistence:
    enabled: true                      # Habilita persistência
    size: 10Gi                        # Tamanho do volume
    storageClass: "longhorn"          # Classe de armazenamento

  resources:                           # Limites de recursos
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 200m
      memory: 256Mi
```

### IngressRoute

```yaml
helm-boilerplate:
  ingressRoutes:
    chartmuseum:
      enabled: true
      entryPoints:
        - web                          # Usando HTTP
      routes:
        - match: Host(`museum.v3m.ai`) # Domínio do ChartMuseum
          services:
            - name: chartmuseum
              port: 8080               # Porta do serviço
```

## Uso

Após a instalação, você pode adicionar o repositório ao Helm:

```bash
helm repo add museum http://museum.v3m.ai
helm repo update
```

Para fazer upload de um chart:

```bash
# Usando curl
curl --data-binary "@mychart-0.1.0.tgz" http://museum.v3m.ai/api/charts

# Ou usando o plugin helm-push
helm plugin install https://github.com/chartmuseum/helm-push
helm push mychart-0.1.0.tgz museum
```

## Monitoramento

O ChartMuseum expõe métricas Prometheus em `/metrics` que podem ser usadas para monitoramento.

## Manutenção

Para atualizar o ChartMuseum:

```bash
helm upgrade chartmuseum ./charts/chartmuseum -n chartmuseum
```

Para desinstalar:

```bash
helm uninstall chartmuseum -n chartmuseum
```

Note que isso não removerá o PVC. Para remover todos os dados:

```bash
kubectl delete pvc -n chartmuseum -l app.kubernetes.io/name=chartmuseum
``` 