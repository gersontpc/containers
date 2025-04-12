### GitHub Workflow

Antes de abordarmos sobre a estratégia de Branch, nós iremos utilizar um mono-repo para contruir uma pipeline para provisionar a infraestrutura, realizar os build/testes de nossa aplicação e por último o deploy. 

Iremos utilizar 2 Branchs principais, uma branch de `infra` que ao realizar o merge nela, irá acionar o workflow (pipeline) de infra, e a branch `main` que será utilizada para realizar o ci/cd de nossa aplicação.

![](./img/github-flow-infra.png)

Os engenheiros irão utilizar `feature` branchs a partir da branch que será utilizada, seja de infra ou aplicação.

### Criando as secrets necessárias

Antes de iniciar o desenvolvimento  da pipeline é necessário criar algumas secrets e variáveis para que workflow faça a criação do cluster ECS na AWS, e preencha esses valores em momento de build.

Secrets a serem criadas: 

```shell
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
AWS_ACCOUNT_ID
```

Variáveis a serem criadas:

```shell
AWS_REGION
```


> O valor das secrets são mascarados na Actions, então fique tranquilo que os valores delas não serão expostos.
> 
> Mascarando da seguinte forma: **"<AWS_ACCOUNT_ID>-tfstate"** para **"\*\*\*-tfstate"** 

Para criar as variáveis no github, vá até o seu repositório importado [lab-ci-cd](lab-ci-cd), clique na aba **Settings**, no menu esquerdo, vá até **Secrets and variables**, **Actions** e clique em **New repository secret**.

![](./img/039.png)

026. Mas antes de criar a secret, vá para o console do AWS Academy, clique em **AWS Details**,  em **AWS CLI**, clique no botão **Show**.

![](./img/040.png)

Copie o valor `aws_access_key_id` .

![](./img/041.png)

Volte para o github onde parou anteriormente e crie a AWS_ACCESS_KEY_ID, e cole o valor que copiou anterior copiado do console AWS Academy, por fim, clique em **Add secret**.

![](./img/042.png)

Pronto, secret criada!

![](./img/043.png)

Agora iremos repetir o mesmo processo para o restante das secrets!

027. Crie a secret AWS_SECRET_ACCESS_KEY, clique no botão **New repository secret**

![](./img/044.png)

Volte no console AWS academy e copie o valor aws_secret_access_key.

![](./img/045.png)

E cole no valor da Secret, e clique em **Add Secret**.

![](./img/046.png)

028. Crie a secret AWS_SESSION_TOKEN, clique no botão **New repository secret**

![](./img/047.png)

Volte no console AWS academy e copie o valor aws_session_token.

![](./img/048.png)

Cole no valor na Secret, e clique em **Add Secret**.

![](./img/049.png)

029. Crie a secret AWS_ACCOUNT_ID, clique no botão **New repository secret**

![](./img/050.png)

Agora volte até o console da AWS, e copie o ID da conta.

![](./img/051.png)

Cole o ID da **Conta** na Secret, e clique em **Add Secret**.

![](./img/054.png)

Pronto! todas as secrets necessárias foram criadas!

> Toda vez que o laboratório for desligado, será necessário reimportar os valores das secrets `AWS_ACCOUNT_ID, AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` e `AWS_SESSION_TOKEN`.

![](./img/055.png)

030. Agora será necessário adicionar qual será a nossa **Região** da AWS que iremos utilizar, mas ao invés de ser uma secret, será uma variável.

Para adicionar a variável **AWS_REGION**, volte para o seu repositório e clique em **Settings**, **Secrets and variables** > **Actions** > Clique na aba **Variables** e por último clique em **New repository variable**, e adicione a variável `AWS_REGION`.

![](./img/0054.png)

031. Volte no console AWS, clique em região e copie a região que está utilizando (us-east-1).

![](./img/0055.png)

032. Volte no GitHub e cole no valor da variável `AWS_REGION`, por último clique em **Add variable**.

![](./img/0056.png)

Pronto! Variável adicionada com sucesso!

![](./img/0057.png)

033. Ainda nas configurações do repositório (**Settings**), no menu esquerdo, clique em **Actions** em seguida **General**.

![](./img/056.png)

034. Desça a página até **Workflow permissions** e altere a permissão para **Read and write permissions** e por último clique em **Save**.

> Essa permissão permite que os workflows tenham permissão de leitura e escrita no repositório, permitindo realizar commits se necessário no repositório.

![](./img/057.png)

### Utilizando o codespaces

Agora iremos utilizar o GitHub CodeSpaces para começar a construir o nosso workflow de infra.

035. Na página inicial do GitHub (https://github.com/), no canto superior direito, clique em  [+], e em **New codespace**.

![](./img/027.png)

036. Preencha as informações, em **Repository** (selecione o repositório USERNAME/container-technologies), **Branch** (selecione a branch *main*), **Region** (selecione a região *US East*) e em **Machine type** (selecione *2-core*), por último clique em **Create codespace**
036. Preencha as informações, em **Repository** (selecione o repositório ), **Branch** (selecione a branch *infra*), **Region** (selecione a região *US East*) e em **Machine type** (selecione *2-core*), por último clique em **Create codespace**

![](./img/028.png)

037. Após criar o codespace, seremos redirecionados para uma [IDE](https://github.com/features/codespaces).

![](./img/029.png)

038. Ao decorrer do nosso laboratório, iremos utilizar o terminal para digitar os comandos necessários para criarmos o nosso workflow.

![](./img/030.png)

### Criando nossa pipeline de infra

039. Seguindo a nossa estratégia de branchs (GitHub Workflow), execute o comando do git para fazer o checkout na branch de infra.

```shell
git checkout infra
```

![](./img/031.png)

040. Execute o comando abaixo para criar os diretórios  `.github/workflows`  e arquivo `infra.yml`  para inserir o código do nosso workflow.

```shell
mkdir -pv .github/workflows
touch .github/workflows/infra.yml
```

![](./img/032.png)

041. Abra o arquivo `infra.yml` e cole o conteúdo abaixo para a criação do nosso workflow.

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

042. Agora iremos executar o comando para criar o nosso diretório de infra e os arquivos necessários para o terraform.

```shell
mkdir -p infra && touch infra/{main.tf,nlb.tf,outputs.tf,sg.tf,variables.tf,versions.tf,terraform.tfvars}
```

Arquivos criados conforme o comando executado!

![](./img/034.png)

043. Agora iremos colocar os nossos "bloquinhos" do terraform de acordo com os arquivos criados.

044. Copie e cole o conteúdo abaixo no arquivo `main.tf`

```terraform
resource "aws_ecs_cluster" "this" {
  name = format("%s-cluster", var.cluster_name)

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

045. Copie o conteúdo abaixo e cole no arquivo `nlb.tf`.

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

046. Copie e cole o conteúdo abaixo no arquivo `sg.tf`.

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

047. Copie e cole o conteúdo abaixo no arquivo `variables.tf`.

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


048. Copie e cole o conteúdo abaixo no arquivo `outputs.tf`.

```terraform
output "load_balancer_arn" {
  value = aws_lb_target_group.this.arn
}

output "nlb_dns_name" {
  value = format("http://%s", aws_lb.this.dns_name)
}
```

049. Copie e cole o conteúdo abaixo no arquivo `versions.tf`.

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

050. Copie e cole o conteúdo abaixo no arquivo `terraform.tfvars`.

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

051. No console da AWS, pesquise por VPC e clique em **VPC** (*Isolated Cloud Resources*).

![](./img/035.png)

052. Em vpc, no menu esquerdo, clique em **Your VPCs** e na coluna **VPC ID** copie o id do VPC e cole na variável `vpc_id` do arquivo terraform.tfvars.

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

053. Ainda em VPC no menu esquerdo, clique em **Subnets** ordene as subnets por **Availability Zone** e copie as subnets `us-east-1a`, `us-east-1b` e `us-east-1c`. 

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

054. Como já preenchemos todos os nossos arquivos do terraform e suas variáveis, iremos criar o último arquivo, o `.gitignore`

Execute o comando abaixo no terminal.

```shell
touch .gitignore
```

055. Copie o conteúdo abaixo e cole no arquivo `.gitignore` .

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

056. Para executarmos o nosso workflow, precisamos criar a feature branch infra `infra`.

![](./img/058.png)

```bash
git checkout -b infra

```

057. Agora iremos commitar as nossas alterações.

```bash
git add -A
git commit -m "chore: create ci/cd infra"
```

![](./img/059.png)

058. Realize o push das alterações.

```bash
git push --set-upstream origin infra
```

![](./img/060.png)


059. Volte para o repositório do github, clique na aba **Pull requests**, e repare que terá um novo pull request: **infra had recent pushes 2 minutes ago**, clique no botão **Compare & pull request**.

![](./img/061.png)

060. Abra o pull request da branch **infra** para branch **infra**.

![](./img/063.png)

061. Faça o merge clicando no botão **Merge pull request**, para mesclar a **infra** para branch **infra**.

![](./img/064.png)

062. Após realizar o merge a *Action* será acionada, clique na aba **Actions**, de clique no workflow **Deploy Infra**, para acompanhar o provisionamento da infra.

![](./img/065.png)

063. Ao clicar em **Deploy infra**, terá a visualização do workflow com o deploy da infra (Cada um dos steps será explicados um a um em sala de aula!), agora clique no step **Terraform Creating and Update**, para visualizar a criação da infra.

![](./img/066.png)

064. Repare que foram criados 5 recursos conforme o output do `terraform apply`.

![](./img/067.png)

065. Agora volte no [console](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1) da AWS para checar se o Cluster ECS, Security Group e NLB foram provisionados conforme esperado.

Abaixo o print com os recursos criados!

ECS > Cluster ECS:
![](./img/068.png)

EC2 >  Network & Security > Cluster ECS:
![](./img/069.png)

EC2 > Load Balancing > Load Balancers
![](./img/070.png)

Show! agora que temos a nossa infraestrutura necessária provisionada iremos seguir para o workflow da nossa app!

066. Volte para o seu repositório e vá até a aba de **Pull requests**, e clique em **New pull request** para sincronizar o código com a branch `main`. 

![](./img/071.png)

067. Faça o pull request da branch `infra` para a branch `main` (Lembrando que é da esquerda para a direita), e por último clique em **Create pull request**.

![](./img/072.png)

068. Clique em **Merge pull request**, para realizar a mesclagem da branch `infra` com a `main`.

![](./img/073.png)

069. E por fim clique em **Confirm merge**, para concluir a mesclagem.

![](./img/074.png)

070. Pronto, merge realizado com sucesso!

![](./img/075.png)