
Como já foi provisionada a infraestrutura pra suportar a nossa aplicação (Cluster ECS, NLB e SG), iremos criar o workflow para `Continuous Integration`, `Continuous Delivery` e `Continuous Deployment`

![](./img/cicd-app.png)

Iremos utilizar os mesmos serviços do GitHub que utilizamos na pipeline de infra.

1. Vá até o GitHub do projeto importado: [https://github.com/**seu-usuario**/template-ci-cd](https://github.com/**<seu-usuario>**/template-ci-cd), abra o Codespaces em seu repositório para trabalhar na construção da pipeline de infra, abra na branch `main` do repositório.

![](./img/001.png)

2. No **terminal** do **Codespaces**, faça crie a branch `feature/init-app`.

[REFERENCIAR NO MARKDOWN O WORKFLOW]

```shell
git checkout -b feature/init-app
```

![](./img/002.png)

3. Após realizada a criação da branch, crie o arquivo `app.yml` para criarmos o workflow do GitHub Actions da nossa aplicação.

```shell
touch .github/workflows/app.yml
```

![](./img/003.png)

4. Clique no arquivo `app.yml` e cole o conteúdo abaixo (workflow) dentro do arquivo.

```yaml
name: 'Deploy App'

on:
  push:
    branches:
      - main

env:
  DESTROY: false
  TF_VERSION: 1.10.5
  IMAGE_NAME: ci-cd-app
  TAG_APP: v1.0.0
  ECS_SERVICE: app-service
  ECS_CLUSTER: app-prod-cluster
  ENVIRONMENT: prod

jobs:
  Build:
    name: 'Building app'
    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: app

    steps:
      - name: Download do Repositório
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install Requirements
        run:  pip install flask

      - name: Unit Test
        run: python -m unittest -v test

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v5
        with:
          fetch-depth: 0
          projectBaseDir: ./app
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build an image from Dockerfile
        env:
          DOCKER_BUILDKIT: 1
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }} .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }}'
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL'

      - name: Push image
        run: |
          docker image push ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }}

  Deploy:

    needs: Build
    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: app

    steps:
      - name: Download do Repositório
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@c804dfbdd57f713b6c079302a4c01db7017a36fc
        with:
          task-definition: ./app/deploy/ecs-task-definition.json
          container-name: ${{ env.IMAGE_NAME }}
          image: ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }}

      - name: Register Task Definition
        id: task-definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}

      - name: Terraform | Setup
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform | Set up statefile S3 Bucket for Backend
        run: |
            echo "terraform {
              backend \"s3\" {
                bucket   = \"${{ secrets.AWS_ACCOUNT_ID }}-tfstate\"
                key      = \"app-${{ env.ENVIRONMENT }}.tfstate\"
                region   = \"${{ vars.AWS_REGION }}\"
              }
            }" >> provider.tf
            cat provider.tf
        working-directory: ./app/deploy

      - name: Terraform | Initialize backend
        run: terraform init
        working-directory: ./app/deploy

      - name: Terraform | Check Syntax IaC Code
        run: terraform validate
        working-directory: ./app/deploy

      - name: Terraform | Plan
        run: terraform plan -out tfplan.binary
        working-directory: ./app/deploy

      - name: Terraform Destroy
        if: env.DESTROY == 'true'
        run: terraform destroy -auto-approve -input=false
        working-directory: ./app/deploy

      - name: Terraform Creating and Update
        if: env.DESTROY != 'true'
        run: terraform apply -auto-approve -input=false
        working-directory: ./app/deploy

      - name: Deploy App in Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@df9643053eda01f169e64a0e60233aacca83799a
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

![](./img/004.png)

> Para mais informações sobre cada step do nosso workflow, [clique aqui!](./github-actions.md)

5. Crie os arquivos essenciais para o nosso projeto.

```shell
mkdir -p app && touch app/{.dockerignore,Dockerfile,app.py,test.py,requirements.txt,sonar-project.properties,docker-compose.yml}
```

![](./img/005.png)

Proposito da criação de cada arquivo:

- `app.py` - Arquivo da aplicação
- `test.py` - Arquivo com o teste unitário
- `requirements.txt` - Arquivo para gerenciar dependências do Python, para mais informações [clique aqui!](https://docs.docker.com/build/concepts/context/#dockerignore-files)
- `.dockerignore` - Arquivo que define quais arquivos e diretórios devem ser ignorados durante o **build** da imagem Docker. Ele funciona de maneira semelhante ao `.gitignore`, impedindo que arquivos desnecessários (como logs, dependências locais, arquivos de configuração sensíveis e diretórios do Git) sejam copiados para a imagem final. Isso ajuda a reduzir o tamanho da imagem e a melhorar a segurança e a eficiência do build. [clique aqui!](https://docs.docker.com/build/concepts/context/#dockerignore-files)
- `Dockerfile` - Arquivo com as instruções para realizar o build da imagem da aplicação, para mais informações [clique aqui!](https://docs.docker.com/build/concepts/context/#dockerignore-files)
- sonar-project.properties - Arquivo para definir configurações do projeto para que o **SonarQube Scanner** faça análise do código corretamente, guia de referência [clique aqui!](https://docs.sonarsource.com/sonarqube-server/10.8/analyzing-source-code/analysis-parameters/)
- deploy.tf - Arquivo que iremos utilizar para realizar o deploy da aplicação na `AWS` onde irá conter os nossos resources do terraform.
- `ecs-task-definition.json` - Definições de configurações para realizar o deploy da nossa aplicação dentro do cluster ECS.

#### Aplicação

A Aplicação é escrita em `python` e utiliza o **Flask** como servidor web.

6. No arquivo `app.py`, copie e cole o conteúdo abaixo:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def pagina_inicial():
    return "Continuous Integration and Continuous Delivery"
```

![](./img/006.png)

7. No arquivo `test.py` copie e cole o conteúdo abaixo.

```python
from app import app
import unittest

class Test(unittest.TestCase):
    def setUp(self):
        # cria uma instância do unittest, precisa do nome "setUp"
        self.app = app.test_client()

        # envia uma requisicao GET para a URL
        self.result = self.app.get('/')

    def test_requisicao(self):
        # compara o status da requisicao (precisa ser igual a 200)
        self.assertEqual(self.result.status_code, 200)

    def test_conteudo(self):
        # verifica o retorno do conteudo da pagina
        self.assertEqual(self.result.data.decode('utf-8'), "Continuous Integration and Continuous Delivery")
```

![](./img/007.png)

8. Para ter um controle melhor das versões de nossas dependências, iremos colar o conteúdo abaixo no arquivo `requirements.txt`.

```txt
flask == 3.1.0
gunicorn == 23.0.0
```

![](./img/008.png)

#### Build

Agora iremos utilizar o **Docker** para relizar a contrução da nossa imagem.

9. No arquivo `.dockerignore` , cope e cole o conteúdo abaixo:

```.dockerignore
Dockerfile
.git
.github
.gitignore
LICENSE
README.md
test.py
./deploy
```

![](./img/009.png)

10. No arquivo `Dockerfile`, copie e cole as informações abaixo.

```Dockerfile
FROM python:3.10-alpine3.21
LABEL MANTAINER="Seu Nome"

WORKDIR /app

COPY . /app

RUN apk add --no-cache curl \
    && pip install --trusted-host pypi.python.org -r requirements.txt

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

![](./img/010.png)

11. No arquivo `docker-compose.yml` cole o conteúdo abaixo.

O `docker-compose.yml` é o arquivo onde passamos as instruções para realizar o build e testar a imagem localmente.

```docker-compose
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: ci-cd-app:v1.0.0
    container_name: ci-cd-app
    restart: always
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=production
```

![](./img/011.png)

#### Testando a aplicação localmente

Agora que incluiu todos os arquivos necessários e seus respectivos conteúdos, você irá testar a app localmente para verificar se está funcionando corretamente. Afinal precisa funcionar localmente (Elimina aquele problema de que só funciona na minha máquina rs).

12. O arquivo `docker-compose.yml`, contém as informações necessárias para subirmos o nosso ambiente localmente.

13. Utilize o comando `docker-compose` para subir a sua aplicação localmente.

```bash
cd app/ && docker compose up
```

Ao digitar o comando, a sua aplicação irá realizar o build da imagem e subir localmente.
Repare que no canto inferior direito, abre sobe uma mensagem `Your application running on port 8000 is available. See all forwarded ports`, clique no botão **Open in Browser**.

![](./img/060.png)

14. Ao clicar no botão, você será redirecionado para uma url que seu aplicativo que está sendo executado local sera exposto.

![](./img/061.png)

Pronto, tudo certo! Seu app está funcionando corretamente localmente! :)

15. Pra parar a execução localmente, pressione `CTRL + C`.

![](./img/062.png)

Agora que testou novamente,  iremos avançar para a configuração do próximo arquivo, mas antes execute o comando para voltar para o diretório `app`.

```shell
cd ..
```

#### Sonar 

Iremos preencher com as informações do SonarQube Cloud, que iremos abordar mais a frente.

16. No arquivo `sonar-project.properties`, copie e cole as informações abaixo:

```sonar
sonar.projectKey=<NOME DO PROJETO>
sonar.organization=<ORGANIZAÇÃO>

# This is the name and version displayed in the SonarCloud UI.
sonar.projectName=<NOME DO PROJETO - Nome do repo> 
#sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
sonar.sources=app.py,Dockerfile

# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8

sonar.language=py
sonar.python.version=3.10

sonar.tests=test.py
sonar.python.coveragePlugin=cobertura
sonar.python.coverage.reportPaths=coverage.xml

sonar.qualitygate.wait=true
```

![](./img/012.png)

#### Deployment

Agora você chegou na fase de adicionar os arquivos do deploy da aplicação.

17. 005. Crie os arquivos essenciais para o deploy da aplicação.

```shell
mkdir -p app/deploy && touch app/deploy/{ecs-task-definition.json,deploy.tf,outputs.tf,terraform.tfvars,variables.tf,versions.tf}
```

![](./img/025.png)

18. No arquivo `ecs-task-definition.json` copie e cole o conteúdo abaixo:

```JSON
{
    "containerDefinitions": [
        {
            "name": "ci-cd-app",
            "image": "gersontpc/ci-cd-app:v1.0.0",
            "cpu": 0,
            "portMappings": [
                {
                    "name": "port-access-8080",
                    "containerPort": 8000,
                    "hostPort": 8000,
                    "protocol": "tcp",
                    "appProtocol": "http"
                }
            ],
            "essential": true,
            "environment": [],
            "environmentFiles": [],
            "mountPoints": [],
            "volumesFrom": [],
            "ulimits": [],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "/ecs/ci-cd-app",
                    "mode": "non-blocking",
                    "awslogs-create-group": "true",
                    "max-buffer-size": "25m",
                    "awslogs-region": "us-east-1",
                    "awslogs-stream-prefix": "ecs"
                },
                "secretOptions": []
            },
            "systemControls": [],
            "healthCheck": {
                "command": ["CMD-SHELL", "curl -f http://0.0.0.0:8000 || exit 1"],
                "interval": 15,
                "timeout": 5,
                "retries": 3,
                "startPeriod": 20
            }
        }
    ],
    "family": "ci-cd-app",
    "taskRoleArn": "arn:aws:iam::526926919628:role/LabRole",
    "executionRoleArn": "arn:aws:iam::526926919628:role/LabRole",
    "networkMode": "awsvpc",
    "volumes": [],
    "placementConstraints": [],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "512",
    "memory": "1024",
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    },
    "tags": [
        {
            "key": "Name",
            "value": "ci-cd-app"
        }
    ]
}
```

![](./img/026.png)

19. No arquivo do Terraform `deploy.tf` copie e cole o conteúdo abaixo:

```hcl
data "aws_lb_target_group" "this" {
  name = "app-prod-tg"
}

data "aws_security_groups" "this" {
  filter {
    name   = "tag:Name"
    values = ["app-prod-sg"]
  }
}

data "aws_lb" "this" {
  name = var.lb_name
}

resource "aws_ecs_service" "this" {
  name                          = "app-service"
  task_definition               = "ci-cd-app"
  cluster                       = var.cluster_name
  desired_count                 = var.desired_count
  launch_type                   = "FARGATE"
  availability_zone_rebalancing = "ENABLED"
  network_configuration {
    subnets          = var.subnets_id
    security_groups  = data.aws_security_groups.this.ids
    assign_public_ip = true
  }

  load_balancer {
    target_group_arn = data.aws_lb_target_group.this.arn
    container_name   = "ci-cd-app"
    container_port   = 8000
  }

}

resource "aws_cloudwatch_log_group" "this" {
  name              = "/ecs/ci-cd-app"
  retention_in_days = 7
}
```

![](./img/027.png)

20. No arquivo `variables.tf` copie e cole o conteúdo abaixo:

```hcl
variable "cluster_name" {
  description = "Nome do cluster ECS"
  type        = string
  default     = "app-prod-cluster"
}

variable "desired_count" {
  description = "desired tasks"
  type        = number
  default     = 3
}

variable "subnets_id" {
  description = "Subnets IDs"
  type        = list(string)
}

variable "lb_name" {
  description = "Load Balancer Name"
  type        = string
  default     = "app-prod-nlb"
}
```

![](./img/028.png)

21. No arquivo `versions.tf`, cope e cole o contaúdo abaixo:

```hcl
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

![](./img/029.png)

22. No arquivo `outputs.tf`, copie e cole o conteúdo abaixo:

```hcl
output "nlb_dns_name" {
  value = format("http://%s", data.aws_lb.this.dns_name)
}
```

![](./img/030.png)

23. No arquivo `terraform.tfvars`, copie e cole o conteúdo abaixo:

```
subnets_id = [
  "subnet-0fbf2767834e01e3c",
  "subnet-0803c20d0114bae14",
  "subnet-06bf008b57618c9c8"
]
```

![](./img/031.png)

Agora que você já criou todos os arquivos necessários, iremos seguir para a criação da conta no `dockerhub`, para realizar o push da imagem do Docker da aplicação.

#### Criação de conta no Dockerhub

Docker Hub é um repositório online de imagens Docker, onde os desenvolvedores podem armazenar, compartilhar e distribuir contêineres. Ele serve como um registro público e privado para armazenar imagens de aplicativos, facilitando a automação do deploy e a colaboração entre equipes.

24. Para criar a conta, acesse https://hub.docker.com/
25. No canto superior esquerdo clique em **Sign up**.

![](./img/015.png)

26. Na página de criação de conta, clique na aba **Personal** (Uso pessoal), em **E-mail** (Insira o seu e-mail), **Username** (Insira seu nome de usuário), **Password** (Insira uma senha) e por último clique em **Sign up**, para criar a sua conta.

![](./img/016.png)

27. Após a criação da conta, será necessário acessar o seu e-mail para verificar a conta do DockerHub, clique em **Verify Email Addres**.

![](./img/019.png)

28. Após clicar em **Verify Email Addres**, será direcionado para a tela informando que seu e-mail foi verificado com sucesso, clique em **Sign In** para realizar o login.

![](./img/020.png)

29. Você será redirecionado para a tela de login, em **Username or email address**, insira o seu nome de usuário ou e-mail, em seguida clique em **Continue**.

![](./img/017.png)

30. Insira a sua senha e clique novamente em **Continue**.

![](./img/018.png)

Ao clicar em **Sign up** você será redirecionado o para o site do Docker, conta criada com sucesso!

![](./img/021.png)

Agora que você já possui a conta, iremos configurar duas secrets no GitHub, no seu projeto.

#### Secrets DockerHub

31. Em seu repositório vá até **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**.

![](./img/022.png)

32.  Em **New secret**, crie a secret `DOCKERHUB_USERNAME` e coloque o username do seu dockerhub.

![](./img/023.png)

33. Repita o processo e crie a secret `DOCKERHUB_TOKEN`, e coloque a sua senha no valor da **Secret**

Logo abaixo, a imagem com as duas secrets criadas.

![](./img/024.png)

#### SonarQube Cloud

O **SonarQube Cloud** é uma plataforma de análise de código na nuvem que ajuda a identificar vulnerabilidades, bugs e problemas de qualidade em aplicações. Ele permite a inspeção contínua do código-fonte, garantindo melhores práticas de desenvolvimento e maior segurança no software.

Será necessária a criação da conta para **analisar a qualidade do código-fonte** utilizando o **SonarQube Cloud**. Esse processo verifica **bugs, vulnerabilidades, code smells e cobertura de testes**, garantindo que o código atenda aos padrões de qualidade antes de ser implantado.

Isso ajuda a melhorar a **segurança, manutenção e desempenho** do projeto, evitando problemas em produção.

34. Acesse o site do [SonarQube Cloud](https://www.sonarsource.com/products/sonarcloud/) e clique em **Login**.

![](./img/032.png)

35. Em seguida clique para realizar o login com o **GITHUB**.

![](./img/033.png)

36. Ao realizar o login clique em **Authorize SonarQubeCloud** para permitir que se autentique no SonarQube com as suas credenciais do GitHub.

![](./img/034.png)

Aguarde o login...

![](./img/035.png)

37. Após o login clique em **Import an organization**.

![](./img/036.png)

38. Em *Install SonarQubeCloud*, selecione a opção **Only select repositories**, selecione o repositório criado em aula ***(sua-conta-do-github)*-lab-ci-cd**, por último clique em **Install**.

![](./img/037.png)

39. Em *Create an organization*, deixe os campos **Nome** e **Key**, padrão, que será o nome do projeto do github que você importou.

Ainda na mesma página, em *Choose a plan*, selecione o plano clicano no botão **Select Free** (Para o plano gratuíto), e clique em **Create Organization** e aguarde a criação do projeto.

![](./img/038.png)
![](./img/039.png)

40. Em *Analyze projects*, selecione o repositório do projeto **lab-ci-cd** e por último clique em **Set Up**.

![](./img/040.png)

41. Em *Set up project for Clean as You Code*, selecione a opção **Previous version** e clique em **Create project**.

![](./img/041.png)

42. No menu esquerdo, clique em **Information** e depois em **Check analysis method**.

![](./img/042.png)

43. Clique em **With GitHub Actions**.

![](./img/043.png)

44. Ao clicar em **With GitHub Actions**, em *1. Disable automatic analysis*, desabilite **Switch off Automatic Analysis**, em *2. Create a GitHub Secret*, terá as credenciais para criar uma nova secret no GitHub Actions, copie o valor do **token** que iremos utilizar posteriormente no valor da secret que irá criar no **GitHub**.

![](./img/044.png)

45. Volte para o repositório, e clique em **Settings** > **Secrets and variables** > **Actions** e clique em **New repository secret**.

![](./img/045.png)

46. Em **Name** coloque o nome da secret `SONAR_TOKEN` e no valor da **Secret** cole o valor do token que copiou no **SonarQube** e clique em **Add secret**.

![](./img/046.png)

47. Secret **SONAR_TOKEN** criada com sucesso!

![](./img/047.png)

48. Volte para o **SonarQube Cloud**, e clique em *Other (for Go, Python, PHP,...)*.

![](./img/048.png)

49. Na sessão *4. Create `sonar.project.properties` file*, copie as informações `sonar.projectKey=<nome-projeto>` e `sonar.organization=<username-github>`, e descomentar os parâmetros `sonar.projectName=<nome-projeto>`, atualize o arquivo **sonar-project.properties** que criou anteriormente.

![](./img/049.png)

50.  O arquivo  **sonar-project.properties**, deve ficar conforme abaixo.

> Altere o `sonar.projectKey`e `sonar.organization`. 

```properties
sonar.projectKey=thiagoqualy2k25_lab-ci-cd
sonar.organization=thiagoqualy2k25

# This is the name and version displayed in the SonarCloud UI.
sonar.projectName=lab-ci-cd
#sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
sonar.sources=app.py,Dockerfile

# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8

sonar.language=py
sonar.python.version=3.10

sonar.tests=test.py
sonar.python.coveragePlugin=cobertura
sonar.python.coverage.reportPaths=coverage.xml

sonar.qualitygate.wait=true
```

![](./img/050.png)

51. Salve o arquivo 


![](./img/051.png)

