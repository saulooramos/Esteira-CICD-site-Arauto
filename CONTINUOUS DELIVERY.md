## CONTINUOUS DELIVERY 

>Building a Tekton Pipeline

Este documento visa passar instruções para construir o Pipeline CI/CD utilizando o Tekton, baseado no Kubernetes. O pipeline exemplificado aqui contém apenas uma task e está previamente configurado no repositório ibm-developer-skills-network/wtecc-CICD_PracticeCode.

>Set Up the Lab Enviroment

Abra o terminal no VScode em Terminal>New Terminal e vá até a pasta do projeto utilizando o comando `cd`:

Exemplo:

`cd /home/project`

Clone o repositório utilizando o comando `git clone`:

Exemplo:

`git clone https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git`

Depois de clonado o repositório, ao abrir o Explorer do VScode, você verá várias pastas e arquivos, lá, procure pelo diretório labs/01_base_pipeline

>step 1: Create an echo Task

De acordo com a tradição de programação, a primeira tarefa que usualmente se cria é um "Hello World!" para aparecer no console.

Há um código já iniciado para os arquivos tasks.yaml e pipeline.yaml conforme a seguir:

>tasks.yaml

`apiVersion: tekton.dev/v1beta1`

`kind: Task`

`metadata:`

`  name: <place-name-here>`

`spec:`

`  steps:`

>pipeline.yaml

`apiVersion: tekton.dev/v1beta1`

`kind: Pipeline`

`metadata:`

`  name: <place-name-here>`

`spec:`

`  tasks:`

Estes arquivos estão de acordo com a estrutura necessária para o Tekton Pipeline.

>Abra o tasks.yaml e mude <place-name-here> para `hello-world`

O próximo passo é adicionar um step, que precisa incluir: name, image, command e args. Neste exemplo, utilizamos:

name: echo

image: alpine:3

command: [/bin/echo]

args: ["Hello Wolrd"]

Assim, o arquivo fica:

`apiVersion: tekton.dev/v1beta1`

`kind: Task`

`metadata:`

`  name: hello-world`

`spec:`

`  steps:`

`    - name: echo`

`       image: alpine:3`
      
`      command: [/bin/echo]`

`      args: ["Hello World!"]`

Depois disso, aplique o arquivo ao clusted kubernetes utilizando o comando `kubectl apply -f tasks.yaml`

>Step 2: Create a hello-pipiline Pipeline

Neste passo, você cria um pipeline muito simples que apenas chama a task hello-world que você criou. Para isto altere o pipeline.yaml de acordo com as instruções abaixo:

<place-name-here>: hello-pipeline

Uma task deve incluir: name e taskRef (que também deve ser atribuída com um name). Neste exemplo utilizamos:

name: hello

taskRef: name: hello-world

Veja que referenciamos a task que criamos no passo anterior. Desta forma o arquivo fica como:

`apiVersion: tekton.dev/v1beta1`

`kind: Pipeline`

`metadata:`

`  name: hello-pipeline`

`spec:`

`  tasks:`

`    - name: hello`

`      taskRef:`

`        name: hello-world`

Depois disso, o próximo passo é aplicar ao cluster Kubernetes utilizando o comando `kubectl apply -f pipeline.yaml`

>Step 3: Run the hello-pipeline

Utilizando o comando `tkn pipeline start --showlog hello-pipeline` você pode testar o pipeline através do Tekton CLI.

Assim, deve aparecer para você a mensagem de Hello World em [hello: echo]

>Step 4: Add a parameter to the task

Agora é hora de criar uma task um pouco mais útil fazendo printar na tela qualquer mensagem que você quiser. Podemos fazer isso adicionando um parâmetro chamado message na task, para isto, renomearemos a task para echo, editando o tasks.yaml conforme abaixo:

`apiVersion: tekton.dev/v1beta1`

`kind: Task`

`metadata:`

`  name: echo`

`spec:`

`params:`
`    - name: message`

`      description: The message to echo`

`      type: string`    

`  steps:`

`    - name: echo-message`

`       image: alpine:3`
      
`      command: [/bin/echo]`

`      args: ["$(params.message)"]`

Depois disso é só utilizar o mesmo comando: kubectl apply -f tasks.yaml

>Step 5: Update the hello-pipeline

Agora você precisa passar a mensagem que você quer enviar à task echo, para isso edite o pipeline.yaml da seguinte forma:

Adicione um params: abaixo de spec: com o name: message.

Mude o name da taskRef para echo.

Adicione um params: na seção de task com um name: message e um value: que referencie o params.message. Assim, o arquivo pipeline.yaml deve ficar:

`apiVersion: tekton.dev/v1beta1`

`kind: Pipeline`

`metadata:`

`  name: hello-pipeline`

`spec:`

`  params:`

`    - name: message`

`  tasks:`

`    - name: hello`

`      taskRef:`

`        name: echo`

`      params:`

`        - name: message`

`          value: "$(params.message)"`

Agora, utilize o comando kubectl apply -f pipeline.yaml para aplicar ao cluster.

Step 6: Run the message-pipeline

Utilize o comando : 

`tkn pipeline start hello-pipeline \`

`    --showlog  \`

`    -p message="Hello Tekton!"`

Deve aparecer para você a mensagem: Hello Tekton!

## Esta primeira parte do laboratório exemplifica muito bem com um passo a passo como funciona o tekton

>Agora é a hora de aplicar utilizando o git

>Step 7: Create a checkout Task

Neste passo você vai combinar o conhecimento de rodar um comando em um container com o de passar parametros para criar uma task que faz o checkout no codigo no Github como o primeiro passo do CD pipeline

Para criar uma checkout task você deve saber que para ter várias definições em um arquivo yaml você deve adicionar 3 hífens (---) em uma mesma linha, desta forma, não precisamos apagar a task que criamos anteriormente. Neste passo vamos adicionar uma nova task ao tasks.yaml que utiliza a imagem do bitnami/git:latest para rodar o comando git ao passar o nome da branch e a url do repositório que você deseja clonar.

Abra o tasks.yaml para criarmos uma nova task com:

name: checkout

params:

1) `name: repourl | type: string | description: "The URL of the git repo to clone"`

2) `name: branch | type: string | description: "The branch to clone"`

`steps: name: checkout | image: bitnamei/git:latest | command: clone | args: ["clone", "--branch", "$(params.branch)", "$(params.repo-url)"]`

Assim, o arquivo tasks.yaml deve ficar:

`---`

`apiVersion: tekton.dev/v1beta1`

`kind: Task`

`metadata:`

`  name: checkout`

`spec:`

`  params:`

`    - name: repo-url`

`      description: The URL of the git repo to clone`

`      type: string`

`    - name: branch`

`      description: The branch to clone`

`      type: string`

`  steps:`

`    - name: checkout`

`      image: bitnami/git:latest`

`      command: [git]`

`      args: ["clone", "--branch", "$(params.branch)", "$(params.repo-url)"]`

É hora de aplicar utilizando o comando kubectl apply -f tasks.yaml

>Step 8: Create the cd-pipeline Pipeline

Finalmente criaremos o pipeline chamado cd-pipeline para ser o ponto de partida do nosso CD pipeline

Abra o aquivo pipeline.yaml para criarmos um novo pipeline chamado cd-pipeline. Novamente, iniciaremos nosso arquivo utilizando três hífens (---) para separarmos o pipeline que criamos anteriormente:

name: cd-pipeline

spec: params: 

1) `name: repo-url`

2) `name: branch | default: "master"`

tasks:

`name: clone | taskRef: name: checkout`

params:

1) `- name: repo-url | value: "$(params.repo-url)"`

2) `- name: branch | value: "$(params.branch)"`

Assim, o arquivo pipeline.yaml deve conter:

`---`

`apiVersion: tekton.dev/v1beta1`

`kind: Pipeline`

`metadata:`

`  name: cd-pipeline`

`spec:`

`  params:`

`    - name: repo-url`

`    - name: branch`

`      default: "master"`

`  tasks:`

`    - name: clone`

`      taskRef:`

`        name: checkout`
        
`      params:`

`      - name: repo-url`

`        value: "$(params.repo-url)"`

`      - name: branch`
      
`        value: "$(params.branch)"`


Aplique ao cluster utilizando o comando: kubectl apply -f pipeline.yaml

>Step 9: Run the cd-pipeline

utilize o comando tkn pipeline start para rodar

Exemplo:

`tkn pipeline start cd-pipeline \`

`    --showlog \`

`    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \`

`    -p branch="main"`

>Step 10: Fill Out cd-pipeline with Placeholders

Neste passo final, você vai preencher com o resto do pipeline com calls à task echo para simplesmente mostrar a imagem por enquanto. No futuro vamos trocar esses placeholders por tasks reais.

Vamos adicionar quatro novas tasks no pipeline: para lint, teste unitário, build e deploy. Estes pipelines vão referenciar a task echo por enquanto.

Nosso código do pipeline-yaml deve ficar conforme abaixo:

```
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cd-pipeline
spec:
  params:
    - name: repo-url
    - name: branch
      default: "master"
  tasks:
    - name: clone
      taskRef:
        name: checkout
      params:
      - name: repo-url
        value: "$(params.repo-url)"
      - name: branch
        value: "$(params.branch)"

    - name: lint
      taskRef:
        name: echo
      params:
      - name: message
        value: "Calling Flake8 linter..."
      runAfter:
        - clone

    - name: tests
      taskRef:
        name: echo
      params:
      - name: message
        value: "Running unit tests with PyUnit..."
      runAfter:
        - lint

    - name: build
      taskRef:
        name: echo
      params:
      - name: message
        value: "Building image for $(params.repo-url) ..."
      runAfter:
        - tests

    - name: deploy
      taskRef:
        name: echo
      params:
      - name: message
        value: "Deploying $(params.branch) branch of $(params.repo-url) ..."
      runAfter:
        - build

```

Novamente, aplicamos ao kubernetes cluster: kubectl apply -f pipeline.yaml

>Step 11: Run the cd-pipeline

Novamente utilizamos o tkn pipeline start.

Exemplo:

`tkn pipeline start cd-pipeline \`

`    --showlog \`

`    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \`

`    -p branch="main"`

Assim, finalizamos esta primeira parte do laboratório, onde nosso output é como se fossem as mensagens de cada etapa do pipeline sendo executadas:

```
PipelineRun started: cd-pipeline-run-wvfzx
Waiting for logs to be available...
[clone : checkout] Cloning into 'wtecc-CICD_PracticeCode'...

[lint : echo-message] Calling Flake8 linter...

[tests : echo-message] Running unit tests with PyUnit...

[build : echo-message] Building image for https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git ...

[deploy : echo-message] Deploying main branch of https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git ...
```
