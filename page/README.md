# Helm Charts - Página de Repositório

Este é o repositório Helm hospedado via GitHub Pages do projeto [m5o-dev/charts](https://github.com/m5o-dev/charts).

## Conteúdo

Esta branch contém:

- Pacotes de charts Helm (arquivos `.tgz`)
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

## Desenvolvimento

**Importante**: Esta branch é gerenciada automaticamente por um workflow GitHub Actions.

Para contribuir com este repositório:
1. Não faça alterações diretamente nesta branch
2. Envie suas alterações de charts para a branch `main` do repositório
3. As alterações serão automaticamente empacotadas e publicadas aqui

Para mais informações, visite o [repositório principal](https://github.com/m5o-dev/charts). 