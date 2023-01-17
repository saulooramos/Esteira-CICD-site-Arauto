## Passo a passo do primeiro laboratório do curso CI/CD Coursera IBM

> Generate GitHub Personal Access Token

Gere o token em Settings>Developer Settings>Personal access tokens>Tokens (classic)

> Fork and Clone the Repository

Vá até a pasta onde você pretende colocar o projeto, faça login com o comando `gh auth login` utilizando o token gerado e utilize o comando:

`gh repo fork <NOMEDOREPOSITORIO> --clone=true`

É importante verificar as permissões dos arquivos do pull request.

>Create a Workflow

Na pasta do projeto, crie um diretório chamado .github/workflows e, dentro dele, um arquivo chamado workflow.yml

`mkdir -p .github/workflows`

`touch .github/workflows/workflow.yml`

>Add Event Triggers

Preencha seu arquivo workflow.yml com a estrutura dos gatilhos dos eventos conforme sua necessidade

Exemplo:

`name: CI workflow`

`on:`

`  push:`

`    branches: [ "main" ]`

`  pull_request:`

`    branches: [ "main" ]`

>Add a job

Ainda no arquivo workflow.yml preencha com os jobs a serem realizados, como build e runner

Exemplo:

`jobs:`

`  build:`

`    runs-on: ubuntu-latest`

>Target the programming language

O CI pipeline deve entender qual a linguagem e versão que será utilizada para compilar seu projeto, sendo assim, insira, ainda no arquivo workflow.yml, a linguagem e versão utilizando o comando `container`:

Exemplo:

`container: python:3.9-slim`

>Save your work

Salve o que você fez no fork do github utilizando os comandos:

`git config --global user.email "you@example.com"`

`git config --global user.name "Your Name"`

`git add -A`

`git commit -m "COMMIT MESSAGE"`

`git push`
