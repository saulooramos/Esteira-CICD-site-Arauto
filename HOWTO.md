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

O CI pipeline deve entender se o projeto deve ser executado em um container docker ou diretamente no SO da VM, sendo assim, especifique se será um projeto dockerizado utilizando o comando  `container`:

Exemplo:

`container: python:3.9-slim`

>Save your work

Salve o que você fez no fork do github utilizando os comandos:

`git config --global user.email "you@example.com"`

`git config --global user.name "Your Name"`

`git add -A`

`git commit -m "COMMIT MESSAGE"`

`git push`

>Step 1: Add the checkout Step

Agora você precisa adicionar o primeiro step no job que você já criou. A primeira tarefa deve ser um checkout (o laboratório recomenda utilizarmos o actions/checkout@v3, uma ação de checkout já pré-definida para realizar essa tarefa) no repositório em questão, faça isto no job build utilizando o seguinte código:

`steps:`
    `- name: Checkout`
    `  uses: actions/checkout@v3` 

>Step 2: Install Dependencies

Agora o próximo passo é instalar as dependências da aplicação. Cada aplicação terá suas próprias dependências, que devem ser instaladas conforme sua necessidade (normalmente em shellscript)

Exempĺo:

`python -m pip install --upgrade pip`

`pip install -r requirements.txt`

Desta forma, inserimos em nosso arquivo workflow.yml mais um step em nosso build, que podemos nomear como "Install dependencies" e utilizamos o comando "run" seguido de dois pontos (:) e uma barra reta caso nosso script contenha mais de um passo.

Exemplo:

`- name: Install dependencies`

`  run:  |`

`   python -m pip install --upgrade pip`

`   pip install -r requirements.txt`

>Step 3: Check your code for syntactical and stylistic issues

Uma ótima maneira de garantir que a qualidade do pipeline CI está adequada é utilizar um serviço de linting, ou seja, de checagem de sintaxe e estilo do código. Alguns exemplos disso são espaçamentos de linhas, identação, localização de variáveis não inicializadas ou indefinidas e falta de parêntesis. o flake8 é uma biblioteca que pode ser utilizada e, em nosso exemplo, é instalada como uma dependência no requirements.txt.

Utilizaremos o comando `flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics`, as opções mencionadas são:

`--count`: Mostra a quantidade de warnings e erros no resultado do comando
`--select=E9,F63,F7,F82`: Limita as checagens a erros de sintaxe
`--show-source`: Adiciona o número da linha onde o erro/warning foi encontrado
`--statistics`: Mostra a contagem de cada tipo de erro e warning no resultado

Como forma de exemplificar, o comando abaixo, diferentemente do primeiro comando mostrado, roda todas as checagens disponíveis do flake8 na pasta do repositório:

`flake8 service --count --max-complexity=10 --max-line-length=127 --statistics`

Sua tarefa agora é adicionar um novo step no "Install dependencies" chamado "Lint with flake8" e adicionar ambos os comandos mencionados para checagem do código.

Exemplo:

`- name: Lint with flake8`
`  run: |`
`   flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics`
`   flake8 service --count --max-complexity=10 --max-line-length=127 --statistics`

>Step 4: Test Code Coverage

Agora, realizaremos testes no código utilizando alguma ferramenta (o laboratório indica para o exemplo a utilização do nosetests, que é configurado dentro de setup.cfg para incluir automaticamente as flags --with-spec e -spec-color para que o red-green-refactor faça sentido, assim, se você estiver utilizando um shell que suporte cores, testes que passarem serão coloridos de verde e testes que falharem de vermelho).

Sua tarefa agora é adicionar mais um step com o nome "Run unit tests with nose" no job build com o seguinte comando:

`nosetests -v --with-spec --spec-color --with-coverage --cover-package=app`

Assim, nosso step ficará desta forma:

`- name: Run unit tests with nose`
`  run: nosetests -v --with-spec --spec-color --with-coverage --cover-package=app`

>Step 5: Push Code to GitHub

Para realizar o teste do workflow e do CI pipeline você precisa commitar as mudanças e fazer um push para o seu branch de volta para o repositório do GitHub. Conforme mencionado, todos os pushs no branch main deve acionar o workflow do pipeline. Para isto, configure sua conta Git utilizando o comando `git config --global user.email` e `user.name` e depois utilize os comandos `git add -A`, `git commit -m "COMMIT MESSAGE"` e `git push`.

Exemplo:

`git config --global user.email saulooramos@gmail.com`

`git config --global user.name saulooramos`

`git add -A`

`git commit -m "COMMIT MESSAGE"`

`git push`

>Step 6: Run the Workflow

Na aba Actions do repositório que você utilizou o fork do GitHub, clique em CI workflow. Ao realizar o push com as mudanças no passo anterior, deve ter triggado uma ação de build e ela estará visível para você. Clique no workflow mais recente para abrir a aba com os detalhes do seu workflow completo. Agora você pode clicar no job build para ver os logs de cada passo e, ao expandir cada seção, você deve visualizar cada uma (atenção à seção do flake8 e dos testes, que se estiverem em verde, estão ok) e, ao final, um "complete job".

>Parabéns!! Agora você está apto a utilizar o GitHub Actions para criar workflows para o pipeline CI!!