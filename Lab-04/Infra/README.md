### Introdução
Neste laboratório, você aprenderá a criar e provisionar a infraestrutura necessária para executar aplicações em um cluster ECS (Elastic Container Service) utilizando o Terraform e o GitHub Actions. O objetivo é automatizar o provisionamento da infraestrutura como código (IaC), garantindo qualidade, segurança e documentação durante o processo.

Ao longo deste laboratório, você irá:

Configurar o ambiente AWS Academy para acessar os recursos necessários.
Criar um bucket S3 para armazenar o statefile do Terraform.
Configurar secrets e variáveis no GitHub para integração com a AWS.
Criar e configurar um workflow no GitHub Actions para provisionar a infraestrutura.
Utilizar o Terraform para criar recursos como clusters ECS, Network Load Balancers (NLB), grupos de segurança e outros componentes essenciais.
Validar o provisionamento da infraestrutura no console da AWS.
Este laboratório é uma oportunidade prática para aplicar conceitos de DevOps, como automação de infraestrutura, controle de versão e integração contínua, utilizando ferramentas amplamente adotadas no mercado. Ao final, você terá uma infraestrutura funcional e pronta para hospedar aplicações em contêineres.



Você irá utilizar 2 Branchs principais, uma branch de `infra` que ao realizar o merge nela, irá acionar o workflow (pipeline) de infra, e a branch `main` que será utilizada para realizar o ci/cd de nossa aplicação (segunda parte deste lab).

Primeiro você irá precisar iniciar o AWS Academy.

## Acessando o AWS Academy

001. Para iniciar o lab, será necessário logar no **AWS Academy**, clicando no link: [clique aqui!](https://www.awsacademy.com/vforcesite/LMS_Login), logo após clique em **Student Login**!

![](./img/001.png)

002. Realize o login na sua conta academica, em **Username**, insira o e-mail da sua conta (A conta que você se cadastrou no AWS Academy), e em **Password**, insira a senha que você cadstrou e por último, clique em **Log In** para acessar a conta.

![](./img/002.png)

003. Após realizar o acesso, no lado esquerdo clique em **Courses**, e em seguida em **All Courses**.

![](./img/003.png)

004. Em **All Courses**, clique no curso liberado pelo professor, **AWS Academy Learner Lab [pegar o ID do curso com o professor]**.

![](./img/004.png)

005. Clique em **Modules**.

![](./img/005.png)

006. Desça até **AWS Academy Learner Lab**, e clique em  **Launch AWS Academy Learner Lab**.

![](./img/006.png)

007. Clique em **Startup Lab**, para iniciar o laboratório (AWS 🔴).

![](./img/007.png)

008. Aguarde alguns minutos até que a aba AWS fique verde (AWS 🟢).

### Acessando o console da AWS

Após ficar verde, clique em cima!

![](./img/008.png)

009. Ao clicar, será redirecionado para o **Console da AWS**.

![](./img/009.png)

010. Pesquise pelo serviço S3 e clique no serviço S3 - *Simple Storage Service*.

![](./img/010.png)

011. Clique em **Create bucket**.

![](./img/011.png)

012. Antes de criarmos o Bucket, precisaremos do ID da conta da AWS.

Vá até o topo do console da AWS e clique na sua conta **voclabs/user2575295=gerson.carneiro...**, em **Account ID** copie o ID da sua conta. 

![](./img/012.png)

013. No serviço **Bucket name**, coloque conforme abaixo (substitua pelo ID da sua conta), ficando da seguinte forma: *526926919628*-tfstate.

Aqui estamos criando o bucket para armazenar o statefile da nossa infraestrutura que será provisionada.

![](./img/013.png)

014. Desça a página e clique em **Create bucket**.

![](./img/014.png)

015. Pronto, bucket criado!

![](./img/015.png)

Agora que o **Bucket** foi criado, crie uma conta no github pois será utilizado alguns dos serviços do github neste laboratório, como: repositório, github actions, codespaces e vscode web.

### Criando uma conta no github

016. Acesse o site do github [https://github.com/](https://github.com/), e clique em **Sign in**.

![](./img/016.png)

017. Se já tiver conta no github, insira o **Username or email address** (e-mail) e **Password** (Senha) e pule para o item [Importando repositório da aula](#importando-repositório-da-aula), depois clique em **Sign in**, caso não tenha conta, vá até **New to GitHub?**, clique em **Create an account** (Criar uma conta).

![](./img/017.png)

018. Em **Sign up to GitHub**, insira o **Email** (E-mail), **Password** (Senha) e **Username** (Nome do usuário), por último clique em **Continue**, para concluir a criação da conta.

![](./img/018.png)

019. Após clicar em continuar, será enviado um código para o e-mail cadastrado para concluir com a criação da conta.

![](./img/019.png)

020. Após inserir o código, você será redirecionado para a página de login.

Repare na mensagem confirmando a criação da sua conta: **Your account was created successfully. Please sign in to continue**

Insira o **e-mail** e **senha** e clique em **Sign in**, para fazer o login!

![](./img/020.png)

021. Pronto conta criada com sucesso!

![](./img/021.png)

Alguns links recomendados para iniciar a jornada no GitHub.
- [GitHub Skills](https://skills.github.com/)
- [Introduction to GitHub](https://github.com/skills/introduction-to-github)
- [Review pull requests](https://github.com/skills/review-pull-requests)
- [Code with Codespaces](https://github.com/skills/code-with-codespaces)
- [Code with Copilot](https://github.com/skills/copilot-codespaces-vscode)
- [Automate workflows with GitHub Actions](https://skills.github.com/#automate-workflows-with-github-actions)

### Importando repositório da aula

022. Agora que você já tem a conta no github, após efetuar o login, vá até o canto superior direito, clique em [+], em seguida clique em **Import repository**.

![](./img/022.png)

> **Import vs fork** 
> - **Import:** A importação geralmente se refere a trazer código de um repositório externo para um repositório existente ou para um novo repositório. Cria uma cópia completa e independente do repositório original em sua própria conta.
> - **Fork:** Um fork cria uma cópia completa e independente do repositório original em sua própria conta. Essa cópia é um repositório separado, com seu próprio histórico de commits e branches. O fork é frequentemente usado para contribuir com projetos de código aberto. Você pode fazer alterações no seu fork e, em seguida, enviar um "pull request" para o repositório original, solicitando que suas alterações sejam incorporadas. O fork oferece isolamento, permitindo que você experimente e faça alterações sem afetar o repositório original.  


023. Em **Import your project to GitHub**, em **Your source repository details** insira o repositório https://github.com/gersontpc/containers-lab, em **Repository name***: insira o nome `container-technologies`, deixe o repositório público selecionando **Public** e por último clique em **Begin Import**.

![](./img/023.png)

Aguarde alguns minutos até o repositório ser importado.

![](./img/024.png)

### Pipeline de infra

![](./img/cicd-infra.png)

Neste primeiro workflow, você irá provisionar automaticamente a infraestrutura de um cluster ECS utilizando o GitHub Actions. Durante o processo, serão aplicadas as ferramentas TFLint, tfsec e terraform-docs para garantir qualidade, segurança e documentação da infraestrutura como código.

Para maiores informações consulte as documentações oficiais:
- [GitHub Actions](https://docs.github.com/pt/actions/writing-workflows/quickstart)
- [TFLint](https://github.com/terraform-linters/tflint)
- [Terraform Docs](https://terraform-docs.io/)
- [ECS - Elastic Container Service](https://docs.aws.amazon.com/pt_br/AmazonECS/latest/developerguide/Welcome.html)


### Criando as secrets necessárias

 Neste momento seu repositório já deve ter sido importado, então será necessário criar algumas secrets e variáveis para que workflow faça a criação do cluster ECS na AWS, essas `secrets` e `variáveis` serão utilizadas workflow para o providionamento da infra.

Secrets necessárias:

```shell
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
AWS_ACCOUNT_ID
```

Variáveis necessárias:

```shell
AWS_REGION
```

> Os valores das secrets são mascarados no momento da execução das Actions, então fiquem tranquilos que os valores delas não serão expostos.
> Mascarando da seguinte forma: **"<AWS_ACCOUNT_ID>-tfstate"** para **"\*\*\*-tfstate"** 

Para criar as variáveis no github, vá até o seu repositório importado [containers-lab](containers-lab), clique na aba **Settings**, no menu esquerdo, vá até **Secrets and variables**, **Actions** e clique em **New repository secret**.

![](./img/039.png)

024. Mas antes de criar a secret, vá para o console do AWS Academy, clique em **AWS Details**,  em **AWS CLI**, clique no botão **Show**.

![](./img/040.png)

Copie o valor `aws_access_key_id` .

![](./img/041.png)

Volte para o github onde parou anteriormente e crie a AWS_ACCESS_KEY_ID, e cole o valor que copiou anterior copiado do console AWS Academy, por fim, clique em **Add secret**.

![](./img/042.png)

Pronto, secret criada!

![](./img/043.png)

Agora repita o mesmo processo para o restante das secrets!

025. Crie a secret AWS_SECRET_ACCESS_KEY, clique no botão **New repository secret**

![](./img/044.png)

Volte no console AWS academy e copie o valor aws_secret_access_key.

![](./img/045.png)

E cole no valor da Secret, e clique em **Add Secret**.

![](./img/046.png)

026. Crie a secret AWS_SESSION_TOKEN, clique no botão **New repository secret**

![](./img/047.png)

Volte no console AWS academy e copie o valor aws_session_token.

![](./img/048.png)

Cole no valor na Secret, e clique em **Add Secret**.

![](./img/049.png)

027. Crie a secret AWS_ACCOUNT_ID, clique no botão **New repository secret**

![](./img/050.png)

Agora volte até o console da AWS, e copie o ID da conta.

![](./img/051.png)

Cole o ID da **Conta** na Secret, e clique em **Add Secret**.

![](./img/054.png)

Pronto! todas as secrets necessárias foram criadas!

> Toda vez que o laboratório for desligado, será necessário reimportar os valores das secrets `AWS_ACCOUNT_ID, AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` e `AWS_SESSION_TOKEN`.

![](./img/055.png)

028. Agora será necessário adicionar a região da AWS que será utilizada **Região**, ao invés de ser uma secret, será uma variável.

Para adicionar a variável **AWS_REGION**, volte para o seu repositório e clique em **Settings**, **Secrets and variables** > **Actions** > Clique na aba **Variables** e por último clique em **New repository variable**, e adicione a variável `AWS_REGION`.

![](./img/0054.png)

029. Volte no console AWS, clique em região e copie a região que está utilizando (us-east-1).

![](./img/0055.png)

030. Volte no GitHub e cole no valor da variável `AWS_REGION`, por último clique em **Add variable**.

![](./img/0056.png)

Pronto! Variável adicionada com sucesso!

![](./img/0057.png)

031. Ainda nas configurações do repositório (**Settings**), no menu esquerdo, clique em **Actions** em seguida **General**.

![](./img/056.png)

032. Desça a página até **Workflow permissions** e altere a permissão para **Read and write permissions** e por último clique em **Save**.

> Essa permissão permite que os workflows tenham permissão de leitura e escrita no repositório, permitindo realizar commits se necessário no repositório.

![](./img/057.png)

### Utilizando o codespaces

Agora será utilizado o GitHub Codespaces para começar a construir o workflow para automatizar o provisionamento da infra.

033. Na página inicial do GitHub (https://github.com/), no canto superior direito, clique em  [+], e em **New codespace**.

![](./img/027.png)

034. Preencha as informações, em **Repository** (selecione o repositório USERNAME/container-technologies), **Branch** (selecione a branch *main*), **Region** (selecione a região *US East*) e em **Machine type** (selecione *2-core*), por último clique em **Create codespace**

035. Preencha as informações, em **Repository** (selecione o repositório ), **Branch** (selecione a branch *infra*), **Region** (selecione a região *US East*) e em **Machine type** (selecione *2-core*), por último clique em **Create codespace**

![](./img/028.png)

036. Após criar o codespace, seremos redirecionados para uma [IDE](https://github.com/features/codespaces).

![](./img/029.png)

037. Ao decorrer do laboratório, será utilizado o terminal para digitar os comandos necessários para criar o workflow.

![](./img/030.png)

### Criando nossa pipeline de infra

038. Seguindo a nossa estratégia de branchs (GitHub Workflow), execute o comando do git para fazer o checkout na branch de infra.

```shell
git checkout infra
```

![](./img/031.png)

039. Execute o comando abaixo para criar os diretórios  `.github/workflows`  e arquivo `infra.yml`  para inserir o código do workflow.

```shell
mkdir -pv .github/workflows
touch .github/workflows/infra.yml
```

![](./img/032.png)

040. Abra o arquivo `infra.yml` e cole o conteúdo abaixo para a criação do workflow.

```yaml
name: 'Deploy Infra'

on:
  push:
    branches:
      - infra
env:
  TF_VERSION: 1.10.5
  TF_LINT_VERSION: v0.52.0
  DESTROY: false
  ENVIRONMENT: prod
jobs:
  terraform:
    name: 'Deploy Infra'
    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: infra

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: AWS | Configure credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ vars.AWS_REGION }}

    - name: Terraform-docs | Generate documentation
      uses: terraform-docs/gh-actions@v1.3.0
      with:
        working-dir: ./infra
        output-file: README.md
        output-method: inject
        git-push: "true"

    - name: Terraform | Check required version
      run: |
        if [ -f versions.tf ];
          then
            echo "TF_VERSION=$(grep required_version versions.tf | sed 's/"//g' | awk '{ print $3 }')" >> $GITHUB_ENV
          else
            echo "Not set required_version in versions.tf, using default version in variable TF_VERSION in file .github/workflows/infra.yml"
            echo "TF_VERSION="${{ env.TF_VERSION }}"" >> $GITHUB_ENV
        fi

    - name: Terraform | Setup
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: ${{ env.TF_VERSION }}

    - name: Terraform | Show version
      run: terraform --version

    - name: Terraform | Set up statefile S3 Bucket for Backend
      run: |
          echo "terraform {
            backend \"s3\" {
              bucket   = \"${{ secrets.AWS_ACCOUNT_ID }}-tfstate\"
              key      = \"infra-${{ env.ENVIRONMENT }}.tfstate\"
              region   = \"${{ vars.AWS_REGION }}\"
            }
          }" >> provider.tf
          cat provider.tf

    - name: Terraform | Initialize backend
      run: terraform init

    - name: Terraform | Format code
      run: terraform fmt

    - name: Terraform | Check Syntax IaC Code
      run: terraform validate

    - name: TFlint | Cache plugin directory
      uses: actions/cache@v4
      with:
        path: ~/.tflint.d/plugins
        key: ${{ matrix.os }}-tflint-${{ hashFiles('.tflint.hcl') }}

    - name: TFlint | Setup TFLint
      uses: terraform-linters/setup-tflint@v4
      with:
        tflint_version: ${{ env.TF_LINT_VERSION }}

    - name: TFlint | Show version
      run: tflint --version

    - name: TFlint | Init TFLint
      run: tflint --init
      env:
        GITHUB_TOKEN: ${{ github.token }}

    - name: TFlint | Run TFLint
      run: tflint -f compact

    - name: TFSec | Security Checks
      uses: aquasecurity/tfsec-action@v1.0.0
      with:
        soft_fail: true

    - name: Terraform | Plan
      run: terraform plan -out tfplan.binary

    - name: Terraform | Show to json file
      run: terraform show -json tfplan.binary > plan.json

    - name: Terraform Destroy
      if: env.DESTROY == 'true'
      run: terraform destroy -auto-approve -input=false

    - name: Terraform Creating and Update
      if: env.DESTROY != 'true'
      run: terraform apply -auto-approve -input=false
```

Ficando da seguinte forma:

![](./img/033.png)

041. Agora execute o comando para criar o diretório de infra e os arquivos necessários para o terraform.

```shell
mkdir -p infra && touch infra/{main.tf,nlb.tf,outputs.tf,sg.tf,variables.tf,versions.tf,terraform.tfvars}
```

Arquivos criados conforme o comando executado!

![](./img/034.png)

042. Agora será colocado os "bloquinhos" do terraform de acordo com os arquivos criados.

043. Copie e cole o conteúdo abaixo no arquivo `main.tf`

```terraform
resource "aws_ecs_cluster" "this" {
  name = format("%s-cluster", var.cluster_name)

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

044. Copie o conteúdo abaixo e cole no arquivo `nlb.tf`.

```terraform
# Crie um Network Load Balancer
resource "aws_lb" "this" {
  name = format("%s-nlb", var.cluster_name)

  subnets            = var.subnets_id
  security_groups    = [aws_security_group.allow_inbound.id]
  load_balancer_type = "network"

  tags = {
    Name = format("%s-nlb", var.cluster_name)
  }
}

# Crie um listener para o NLB
resource "aws_lb_listener" "this" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "TCP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}

# Crie um target group
resource "aws_lb_target_group" "this" {
  name        = format("%s-tg", var.cluster_name)
  port        = 80
  protocol    = "TCP"
  vpc_id      = var.vpc_id
  target_type = "ip"

  tags = {
    Name = format("%s-tg", var.cluster_name)
  }
}
```

045. Copie e cole o conteúdo abaixo no arquivo `sg.tf`.

```terraform
# Crie um grupo de segurança
resource "aws_security_group" "allow_inbound" {
  name        = format("%s-sg", var.cluster_name)
  vpc_id      = var.vpc_id
  description = "Allow inbound traffic"

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = format("%s-sg", var.cluster_name)
  }
}


resource "aws_vpc_security_group_ingress_rule" "web" {
  security_group_id = aws_security_group.allow_inbound.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  ip_protocol = "tcp"
  to_port     = 80
  description = "Acesso Web"
}

resource "aws_vpc_security_group_ingress_rule" "container" {
  security_group_id = aws_security_group.allow_inbound.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 8000
  ip_protocol = "tcp"
  to_port     = 8000
  description = "Acesso Web"
}

```

046. Copie e cole o conteúdo abaixo no arquivo `variables.tf`.

```terraform
variable "cluster_name" {
  description = "Nome do cluster ECS"
  type        = string
}

variable "subnets_id" {
  description = "Subnets IDs"
  type        = list(string)
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}
```


047. Copie e cole o conteúdo abaixo no arquivo `outputs.tf`.

```terraform
output "load_balancer_arn" {
  value = aws_lb_target_group.this.arn
}

output "nlb_dns_name" {
  value = format("http://%s", aws_lb.this.dns_name)
}
```

048. Copie e cole o conteúdo abaixo no arquivo `versions.tf`.

```terraform
terraform {
  required_version = "1.10.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.86.0"
    }
  }
}
```

049. Copie e cole o conteúdo abaixo no arquivo `terraform.tfvars`.

```terraform
cluster_name = "app-prod"

vpc_id = ""

subnets_id = [
  "",
  "",
  ""
]
```

Após colar o conteúdo do arquivo `terraform.tfvars` precisamos preencher as variáveis `vpc_id` e `subnets_id`.

Será necessário acessar o console da AWS para pegar os IDs das Subnets e VPC.

050. No console da AWS, pesquise por VPC e clique em **VPC** (*Isolated Cloud Resources*).

![](./img/035.png)

051. Em vpc, no menu esquerdo, clique em **Your VPCs** e na coluna **VPC ID** copie o id do VPC e cole na variável `vpc_id` do arquivo terraform.tfvars.

![](./img/036.png)

```terraform
cluster_name = "app-prod"

vpc_id = "vpc-0adcac6bf7f8c2e7f"

subnets_id = [
  "",
  "",
  ""
]
```

052. Ainda em VPC no menu esquerdo, clique em **Subnets** ordene as subnets por **Availability Zone** e copie as subnets `us-east-1a`, `us-east-1b` e `us-east-1c`. 

![](./img/037.png)

```terraform
cluster_name = "app-prod"

vpc_id = "vpc-0adcac6bf7f8c2e7f"

subnets_id = [
  "subnet-0fbf2767834e01e3c",
  "subnet-0803c20d0114bae14",
  "subnet-06bf008b57618c9c8"
]
```

053. Como já preenchido todos os arquivos do terraform e suas variáveis, crie o último arquivo, o `.gitignore`.

Execute o comando abaixo no terminal.

```shell
touch .gitignore
```

054. Copie o conteúdo abaixo e cole no arquivo `.gitignore` .

```gitignore
### Git ###
# Created by git for backups. To disable backups in Git:
# $ git config --global mergetool.keepBackup false
*.orig

# Created by git when using merge tools for conflicts
*.BACKUP.*
*.BASE.*
*.LOCAL.*
*.REMOTE.*
*_BACKUP_*.txt
*_BASE_*.txt
*_LOCAL_*.txt
*_REMOTE_*.txt

### Terraform ###
# Local .terraform directories
**/.terraform/*
.terraform.lock.hcl
# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files, which are likely to contain sensitive data, such as
# password, private keys, and other secrets. These should not be part of version
# control as they are data points which are potentially sensitive and subject
# to change depending on the environment.
# *.tfvars
*.tfvars.json

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Include override files you do wish to add to version control using negated pattern
# !example_override.tf

# Include tfplan files to ignore the plan output of command: terraform plan -out=tfplan
# example: *tfplan*

# Ignore CLI configuration files
.terraformrc
terraform.rc
```

![](./img/038.png)

055. Para executar o workflow, será necessário commitar as nossas alterações.

```bash
git add -A
git commit -m "chore: create workflow and infra ECS"
```

![](./img/0059.png)

056. Realize o push das alterações.

```bash
git push
```

![](./img/060.png)


057. Após realizar o push a *Action* será acionada, clique na aba **Actions**, de clique no workflow **Deploy Infra**, para acompanhar o provisionamento da infra.

![](./img/065.png)

058. Ao clicar em **Deploy infra**, terá a visualização do workflow com o deploy da infra (Cada um dos steps será explicados um a um em sala de aula!), agora clique no step **Terraform Creating and Update**, para visualizar a criação da infra.

![](./img/066.png)

059. Repare que foram criados 5 recursos conforme o output do `terraform apply`.

![](./img/067.png)

060. Agora volte no [console](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1) da AWS para checar se o Cluster ECS, Security Group e NLB foram provisionados conforme esperado.

Abaixo o print com os recursos criados!

ECS > Cluster ECS:
![](./img/068.png)

EC2 >  Network & Security > Cluster ECS:
![](./img/069.png)

EC2 > Load Balancing > Load Balancers
![](./img/070.png)

061. Por último faça o Pull Request da branch `infra` para a `main` e realizer o merge.

No repositório clique na aba **Pull requests** e em seguida **New pull request**

![](./img/071.png)

062. Selecione a branch de infra para realizar o merge na main, ficando da sequinte forma [from:main] <- [compare:infra], depois clique em **Create pull request**.

![](./img/072.png)

063. Clique **Create pull request**

![](./img/073.png)

064. Clique em **Merge pull request** para realizar o merge!

![](./img/074.png)

Show! Laboratório concluído! 🎉
Agora que você já tem a infraestrutura necessária provisionada de forma automatizada, ela está pronta para receber o deploy da aplicação, que será executado na continuação do laboratório:[Deploy App](../App/README.md).
