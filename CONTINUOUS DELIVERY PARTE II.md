## CONTINUOUS DELIVERY PARTE II

>Neste segundo laboratório, iremos adicionar triggers Github ao criar um EventListener, um TriggerBinding e um TriggerTemplate para o Tekton

>Primeiro, confiture o ambiente para que você possa começar a trabalhar, clonando o repositório em questão e abrindo o diretório onde os seus arquivos eventlistener,yaml, pipeline.yaml, tasks.yaml, triggerbinding.yaml e triggertemplate.yaml estão.

Exemplo:
```
cd /home/project
git clone https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git
cd wtecc-CICD_PracticeCode/labs/02_add_git_trigger/
```
Para continuar, você precisa já ter estruturado os arquivos tasks.yaml e pipeline.yaml, que foram escritos no laboratório anterior. Assim, instalamos os arquivos com os comandos:

```
kubectl apply -f tasks.yaml
kubectl apply -f pipeline.yaml
```

Você pode checar as tasks criadas com o comando `tkn task ls` e o pipeline criado com `tkn pipeline ls`

>Step 1: Create an EventListener

O eventlistener vai escutar incoming events do GitHub. Para que ele funcione, escreva no arquivo eventlistener.yaml definindo:

```
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: cd-listener
spec:
  serviceAccountName: pipeline
  triggers:
    - bindings:
      - ref: cd-binding
      template:
        ref: cd-template
```

Agora, aplique o EventListener no cluster kubernetes utilizando o comando: `kubectl apply -f eventlistener.yaml`

Você pode verificar o que foi realizado utilizando o comando `tkn eventlistener ls`

>Step 2: Create a TriggerBinding

O próximo passo é criar um triggerbinding, escrevendo o arquivo triggerbinding.yaml ao definir:

```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: cd-binding
spec:
  params:
    - name: repository
      value: $(body.repository.url)
    - name: branch
      value: $(body.ref)
```

Observe que o nome do TriggerBinding é o mesmo referenciado no EventListener. Além disso, referenciamos o repositório como $(body.repository.url) e a branch como $(body.ref)

Depois disso, é só aplicar no cluster com o comando `kubectl apply -f triggerbinding.yaml`

>Step 3: Create a TriggerTemplate

Agora chegou a hora de especificar o arquivo triggertemplate.yaml:

```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: cd-template
spec:
  params:
    - name: repository
      description: The git repo
      default: " "
    - name: branch
      description: the branch for the git repo
      default: master
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: cd-pipeline-run-
      spec:
        serviceAccountName: pipeline
        pipelineRef:
          name: cd-pipeline
        params:
          - name: repo-url
            value: $(tt.params.repository)
          - name: branch
            value: $(tt.params.branch)
```

Observe que o TriggerTemplate tem o mesmo nome referenciado no EventListener. Além disso, referenciamos o repositório como $(tt.params.repository) e a branch como $(tt.params.branch), de forma que o tt referencia o triggertemplate.

Depois disso, aplicamos no cluster com o comando `kubectl apply -f triggertemplate.yaml`

>Step 4: Start a Pipeline Run

Agora, podemos chamar o eventlistener para iniciar um Pipeline Run. Você pode realizar isso localmente utilizando o comando `curl`.

O primeiro passo é rodar o comando `kubectl port-forward` para chamar o eventlistener no localhost, utilizando as portas 8090:8090.

`kubectl port-forward service/el-cd-listener  8090:8080`

Abra um novo terminal para triggar o eventlistener postando no endpoint que você esta escutando através do comando curl:

```
curl -X POST http://localhost:8090 -H 'Content-Type: application/json' -d '{"ref":"main","repository":{"url":"https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode"}}'
```

Isso deve dar start no seu pipeline Run

## UTILIZANDO O TEKTON CATALOG

>Agora, vamos utilizar o Tekton Catalog como referência para realizar um git-clone task para clonar o repositório Git.

Neste laboratório, já clonamos o repositório que consta a pasta 03_use_tekton_catalog, com os arquivos que vamos referenciar nos passos a seguir. Para isso, abrimos a pasta com o comando `cd wtecc-CICD_PracticeCode/labs/03_use_tekton_catalog/`

>Step 1: Add the git-clone Task

Começamos utilizando o tekton CLI para aplicar o arquivo task git-clone com o comando: `tkn hub install task git-clone --version 0.8`

Este comando instala o git-clone task no cluster.

>Step 2: Create a Workspace

Agora, vamos utilizar o arquivo pvc.yaml que referencia nosso volume persistente:

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cd-pipeline
spec:
  workspaces:
    - name: pipeline-workspace
  params:
    - name: repo-url
    - name: branch
      default: "master"
  tasks:
    - name: init
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: cleanup

    - name: clone
      workspaces:
        - name: output
          workspace: pipeline-workspace
      taskRef:
        name: git-clone
      params:
      - name: url
        value: $(params.repo-url)
      - name: revision
        value: $(params.branch)
      runAfter:
        - init

    # Note: The remaining tasks are unchanged. Do not delete them.
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

Agora, aplicamos ao cluster com o comando `kubectl apply -f pipeline.yaml` e estamos prontos para rodar o pipeline

>Step 4: Run the Pipeline

Utilizamos o código:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

Pronto!! Você utilizou o git-clone do tekton catalog ao invés de escrever tudo você mesmo! Sempre recorra ao catalogo antes de criar tudo sozinho.
