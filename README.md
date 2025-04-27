# Helm Charts Repository

Este repositório contém uma coleção de charts Helm personalizados, disponibilizados através do GitHub Pages. Os charts são automaticamente empacotados e publicados usando GitHub Actions.

## Como Funciona

O fluxo de trabalho deste repositório:

1. Os charts Helm são armazenados na branch `main`
2. A branch `gh-pages` contém apenas os charts empacotados e o arquivo `index.yaml`
3. Quando há um merge para a branch `main`, uma ação do GitHub é disparada
4. Esta ação empacota os charts e atualiza a branch `gh-pages`

## Utilização

Para adicionar este repositório ao seu Helm:

```bash
helm repo add m5o-charts https://m5o.dev/charts
helm repo update
```

Você pode então instalar os charts disponíveis:

```bash
helm install nome-da-release m5o-charts/nome-do-chart
```

## Documentação

Consulte o arquivo [DOCUMENTATION.md](DOCUMENTATION.md) para um guia detalhado sobre como criar seu próprio repositório de charts Helm usando GitHub Pages e GitHub Actions.
