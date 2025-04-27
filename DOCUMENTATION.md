# Criando um Repositório de Charts Helm com GitHub Pages

Este guia descreve como configurar seu próprio repositório de charts Helm usando GitHub Pages para hospedagem e GitHub Actions para automação da publicação.

## Pré-requisitos

- Uma conta no GitHub
- [Git](https://git-scm.com/) instalado em sua máquina
- [Helm](https://helm.sh/docs/intro/install/) instalado em sua máquina
- Conhecimento básico de Helm e GitHub

## Passo 1: Configurar o Repositório GitHub

1. Crie um novo repositório no GitHub
2. Clone o repositório para sua máquina local:
   ```bash
   git clone https://github.com/seu-usuario/seu-repositorio.git
   cd seu-repositorio
   ```
3. Crie uma estrutura básica para seu projeto:
   ```bash
   mkdir -p charts
   touch README.md
   ```

## Passo 2: Criar um Chart Helm

1. Dentro do diretório `charts`, crie seu primeiro chart:
   ```bash
   helm create charts/meu-primeiro-chart
   ```
2. Personalize o chart conforme necessário, editando os arquivos dentro do diretório do chart

## Passo 3: Configurar o GitHub Pages

1. Nas configurações do seu repositório GitHub, ative o GitHub Pages
2. Configure-o para usar a branch `gh-pages` como origem (você não precisa criar esta branch manualmente, a GitHub Action irá criá-la)

## Passo 4: Configurar a GitHub Action

1. Crie o diretório `.github/workflows/` no seu repositório:
   ```bash
   mkdir -p .github/workflows
   ```

2. Crie um arquivo chamado `.github/workflows/release.yml` com o seguinte conteúdo:

```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Install Helm
        uses: azure/setup-helm@v3

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.5.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

## Passo 5: Fazer Push e Verificar

1. Commit e push das alterações para a branch `main`:
   ```bash
   git add .
   git commit -m "Configuração inicial do repositório de charts"
   git push origin main
   ```

2. Verifique a aba "Actions" no GitHub para acompanhar a execução do workflow
3. Após a conclusão bem-sucedida, a branch `gh-pages` será criada com os charts empacotados

## Passo 6: Usar o Repositório

Agora você pode adicionar seu repositório Helm aos seus repositórios locais:

```bash
helm repo add meu-repo https://seu-usuario.github.io/seu-repositorio
helm repo update
```

E instalar charts a partir dele:

```bash
helm install minha-release meu-repo/meu-primeiro-chart
```

## Adicionando Novos Charts

Para adicionar novos charts ao repositório:

1. Crie o chart dentro do diretório `charts/`
2. Faça commit e push para a branch `main`
3. A GitHub Action será automaticamente disparada, empacotando e publicando o novo chart

## Atualizando Charts Existentes

Para atualizar um chart existente:

1. Modifique os arquivos no diretório do chart
2. **Importante**: Atualize a versão no arquivo `Chart.yaml` do chart
3. Faça commit e push para a branch `main`
4. A GitHub Action será automaticamente disparada, atualizando o chart na branch `gh-pages`

## Dicas e Boas Práticas

- Mantenha sua estrutura de diretórios organizada
- Sempre atualize a versão no arquivo `Chart.yaml` ao modificar um chart
- Documente bem seus charts no arquivo `README.md` dentro do diretório de cada chart
- Use testes para validar seus charts antes de publicá-los
- Considere adicionar um fluxo de revisão de código usando Pull Requests

## Solução de Problemas

### A GitHub Action falha ao empacotar os charts

- Verifique se a versão no arquivo `Chart.yaml` foi incrementada
- Certifique-se de que todos os dependências estão disponíveis
- Verifique os logs da Action no GitHub para identificar erros específicos

### O chart não aparece no repositório

- Verifique se o chart está no diretório correto (`charts/`)
- Certifique-se de que a GitHub Action foi executada com sucesso
- Execute `helm repo update` para atualizar seu cache local

## Recursos Adicionais

- [Helm Charts - Documentação Oficial](https://helm.sh/docs/topics/charts/)
- [GitHub Actions - Documentação](https://docs.github.com/pt/actions)
- [chart-releaser-action - GitHub](https://github.com/helm/chart-releaser-action) 