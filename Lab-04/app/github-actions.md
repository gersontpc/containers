### GitHub Actions Build/Deploy App

Abaixo uma explicação de cada **step** desse **GitHub Actions** para build da app com **Terraform**:

1. Configuração Geral

- **Nome do workflow:** `Deploy App`.
- **Disparo do workflow:** Sempre que houver um **push** na branch `main`.
- Define variáveis de ambiente (global):
	- `DESTROY`: Define se o Terraform deve destruir os recursos (`false` por padrão).
	- `TF_VERSION`: Versão do Terraform utilizada.
	- `IMAGE_NAME`: Nome da imagem Docker.
	- `TAG_APP`: Tag da imagem (versão `v1.0.0`).
	- `ECS_SERVICE`: Nome do serviço ECS.
	- `ECS_CLUSTER`: Nome do cluster ECS.
	- `ENVIRONMENT`: Ambiente (`prod`).

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
```

---
#### Build app

2. Definição do Job `Building app`

- O job **`Build`** roda na máquina **`ubuntu-latest`**.
- Define que os comandos executados no shell utilizarão o `bash` dentro do diretório **`app`**.

```yaml
jobs:
  Build:
    name: 'Building app'
    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: app
```

---

3. Checkout do Código

-   Faz o **checkout** do código-fonte no runner.
-   O `fetch-depth: 0` garante que **todo o histórico do Git** seja baixado.

```yaml
  - name: Download do Repositório
	uses: actions/checkout@v4
	with:
	  fetch-depth: 0
```

---

4. Configuração do Python

-   Configura o ambiente Python **versão 3.10**

```yaml
  - name: Setup Python
	uses: actions/setup-python@v4
	with:
	  python-version: '3.10'
```

---

5. Instalação das Dependências

-   Instala a **dependência Flask** para a aplicação.
```yaml
- name: Install Requirements
  run:  pip install flask -r requirements.txt
```

---

6. Executando Testes Unitários**

- Roda os **testes unitários** com o **`unittest`**.

```yaml
  - name: Unit Test
	run: python -m unittest -v test
```

---

7. Análise Estática com SonarQube

- **Executa uma análise de qualidade de código** no **SonarQube**.
- Usa o **token secreto `SONAR_TOKEN`** armazenado no GitHub Secrets.

```yaml
  - name: SonarQube Scan
	uses: SonarSource/sonarqube-scan-action@v5
	with:
	  fetch-depth: 0
	  projectBaseDir: ./app
	env:
	  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```
---

8. Login no Docker Hub

- Faz login no **Docker Hub** usando credenciais secretas.

```yaml
  - name: Login to Docker Hub
	uses: docker/login-action@v3
	with:
	  username: ${{ secrets.DOCKERHUB_USERNAME }}
	  password: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

9. Construindo a Imagem Docker

-   **Habilita o BuildKit (`DOCKER_BUILDKIT=1`)** para uma compilação mais eficiente.
-   Constrói a **imagem Docker** com a tag definida (`IMAGE_NAME:TAG_APP`).

```yaml
  - name: Build an image from Dockerfile
	env:
	  DOCKER_BUILDKIT: 1
	run: |
	  docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }} .
```

---

10. Scanner de Vulnerabilidades com Trivy

-   **Escaneia a imagem Docker** em busca de vulnerabilidades.
-   Se houver vulnerabilidades **CRÍTICAS**, o workflow **falha (`exit-code: 1`)**.

```yaml
  - name: Run Trivy vulnerability scanner
	uses: aquasecurity/trivy-action@master
	with:
	  image-ref: '${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }}'
	  format: 'table'
	  exit-code: '1'
	  ignore-unfixed: true
	  vuln-type: 'os,library'
	  severity: 'CRITICAL'
```
---

11. Publicando a Imagem no Docker Hub

-   **Faz o push da imagem** para o **Docker Hub**.

```yaml
  - name: Push image
	run: |
	  docker image push ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.TAG_APP }}
```
---

#### Build Deploy

- **Executado após o `Build` (`needs: Build`)**.
- **Responsável por:** Configurar credenciais AWS, atualizar task definition, criar infra com Terraform e fazer deploy no ECS.

12. Checkout do Código

-   Faz o **checkout** do código-fonte no runner.
-   O `fetch-depth: 0` garante que **todo o histórico do Git** seja baixado.

```yaml
  - name: Download do Repositório
	uses: actions/checkout@v4
	with:
	  fetch-depth: 0
```

13. Configurar credenciais AWS

- Configura credenciais para usar AWS CLI.

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
    aws-region: ${{ vars.AWS_REGION }}
```

14. Atualizar Task Definition

- - Substitui a imagem no arquivo **`ecs-task-definition.json`**.

```yaml
- name: Register Task Definition
  id: task-definition
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
```

15. Registrar Task Definition

- Registra a nova **task definition** no ECS.

```yaml
- name: Register Task Definition
  id: task-definition
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
```

16. Configurar Terraform

- Instala o **Terraform** na versão especificada.

```yaml
- name: Terraform | Setup
  uses: hashicorp/setup-terraform@v3
  with:
    terraform_version: ${{ env.TF_VERSION }}
```

17. Configurar Backend S3

- Define o **Backend do Terraform no S3**.

```yaml
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
```

18. Inicializar Terraform

- Inicializa o Terraform e baixa os plugins.

```yaml
- name: Terraform | Initialize backend
  run: terraform init
  working-directory: ./app/deploy
```

19. Validar código Terraform

- Verifica erros de sintaxe no código Terraform.

```yaml
- name: Terraform | Check Syntax IaC Code
  run: terraform validate
  working-directory: ./app/deploy
```

20. Criar plano de execução

- Gera o plano de mudanças.

```yaml
- name: Terraform Destroy
  if: env.DESTROY == 'true'
  run: terraform destroy -auto-approve -input=false
  working-directory: ./app/deploy
```

21. Destruir Infraestrutura (se necessário)

- Executa `terraform destroy` se `DESTROY=true`.

```yaml
- name: Terraform Creating and Update
  if: env.DESTROY != 'true'
  run: terraform apply -auto-approve -input=false
  working-directory: ./app/deploy
```

22. Aplica as configurações no **AWS ECS**

- **Faz o deploy** no **Amazon ECS**.

```yaml
- name: Deploy App in Amazon ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@df9643053eda01f169e64a0e60233aacca83799a
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
    service: ${{ env.ECS_SERVICE }}
    cluster: ${{ env.ECS_CLUSTER }}
    wait-for-service-stability: true
```

## **Conclusão**

Esse workflow faz **build, análise, testes, push de imagem e deploy automático no ECS** utilizando **GitHub Actions, Docker e Terraform**. 🚀