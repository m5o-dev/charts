# Helm Charts - Página de Repositório ;)

Este é o repositório Helm hospedado via GitHub Pages do projeto [m5o-dev/charts](https://github.com/m5o-dev/charts).

## Conteúdo

Esta branch contém:

- Pacotes de charts Helm organizados em diretórios dedicados (`charts/{nome-do-chart}/*.tgz`)
- Arquivo `index.yaml` para indexação dos charts
- Este README explicativo

## Uso

Para adicionar este repositório Helm ao seu ambiente:

```bash
helm repo add m5o-charts https://m5o-dev.github.io/charts/
helm repo update
```

Para listar os charts disponíveis:

```bash
helm search repo m5o-charts
```

Para instalar um chart específico:

```bash
helm install my-release m5o-charts/{nome-do-chart}
```

## Estrutura do Repositório

Os charts Helm são organizados da seguinte maneira:

```
/charts
  /{nome-do-chart-1}
    /{nome-do-chart-1}-1.0.0.tgz
    /{nome-do-chart-1}-1.0.1.tgz
  /{nome-do-chart-2}
    /{nome-do-chart-2}-2.0.0.tgz
```

Isso permite manter múltiplas versões de cada chart no repositório.

## Desenvolvimento

**Importante**: Esta branch é gerenciada automaticamente por um workflow GitHub Actions.

Para contribuir com este repositório:
1. Não faça alterações diretamente nesta branch
2. Envie suas alterações de charts para a branch `main` do repositório
3. As alterações serão automaticamente empacotadas e publicadas aqui

Para mais informações, visite o [repositório principal](https://github.com/m5o-dev/charts). 