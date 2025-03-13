## Aula 3 - Container Technologies

### Preparando o ambiente no Cloud9

1. Criando o ambiente no Cloud9

Vá até serviços e pesquise por `Cloud9`

![cloud9](img/cloud9-00.png)

2. Em Cloud9 crie em `Create environment`:

![cloud9](img/cloud9-01.png)

3. Preencha os campos conforme as instruções abaixo:

- **Details:**  
**Name:** container-technologies  
**Environment type:** New EC2 instance  

- **New EC2 instance:**  
Selecione **Instance Type:** t3.small (2GiB RAM + 2vCPU)  
**Plataform:** Ubuntu Server 22.04 LTS  

- **Network settings**  
Selecione: **Secure Shell (SSH)**

![cloud9](img/cloud9-02.png)

4. E por último clique em **Create**

![cloud9](img/cloud9-04.png)

5. Aguarde a criação do ambiente

![cloud9](img/cloud9-05.png)

6. Enquanto cria o ambiente, vá no serviço EC2:

![cloud9](img/cloud9-06.png)

7. Selecione a instância criada `aws-cloud9-container-technologies-<ID/>`:
- Ao selecionar a instância, desça até a final da página e clique na aba **Security** e depois em  **Security Groups** clique no Segurity Group da instância.


![cloud9](img/cloud9-07.png)

8. No Security Group clique em **Edit inbound rules:**


![cloud9](img/cloud9-08.png)

9. Clique em **Save rules**, para salvar a regra criada

![cloud9](img/cloud9-09.png)

10. Vá novamente até o **Cloud9**, e clique em **Open**:

![cloud9](img/cloud9-10.png)

11. Conhecendo o **Cloud9**:

![cloud9](img/cloud9-11.png)

12. Clonando o repositório da aula

No terminal do Cloud9, execute o comando abaixo:

```shell
git clone https://github.com/gersontpc/container-technologies.git
```

### Exercicio 3 - docker compose (Wordpress)

Após clonar o repositório, iremos acessar o diretório 

13. Acesse o diretório Lab-03:
```shell
cd container-technologies/Lab-03/
```

14. Execute o comando `docker compose up -d` para iniciar os serviços do wordpress

```shell
docker compose up -d
```
![cloud9](img/compose-01.png)

Conteúdo do **compose.yaml**:
```
version: "3.8"

services:
  mysql:
    image: mariadb:latest
    command: '--default-authentication-plugin=mysql_native_password'
    deploy:
      replicas: 1
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: senha1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: UserBlog
      MYSQL_PASSWORD: PwdBlog
    expose:
      - 3306
    volumes:
      - database:/var/lib/mysql
    networks:
      - wordpress

  wordpress:
    image: wordpress:latest
    deploy:
      replicas: 1
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: UserBlog
      WORDPRESS_DB_PASSWORD: PwdBlog
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
    ports:
      - 8080:80
    networks:
      - wordpress
    depends_on:
      - mysql

volumes:
  database:
  wordpress:

networks:
  wordpress:
    driver: bridge

```
15. Após subir a stack do wordpress, execute o comando abaixo


```shell
URL="http://$(aws ec2 describe-instances \
    --instance-ids $(curl -s http://169.254.169.254/latest/meta-data/instance-id) \
    --query "Reservations[0].Instances[0].PublicIpAddress" --output text):8080" && echo $URL
```
Output:
```shellscript
http://44.202.53.105:8080
```

16. Copie a URL e cole em seu navegador para abrir o frontend do wordpress

Selecione **Português do Brasil** e clique em **Continuar**

![cloud9](img/wordpress-01.png)

17. Preencha a tela de boas vindas  
**Título do site:** container-technologies  
**Nome do usuário:** wpuser  
**O seu e-mail:** seu e-mail da faculdade <e-mail>  
Em seguida clique em **Instalar Wordpress**

18. Na tela de login coloque o usuário *wpuser* e a senha gerada.

**Nome de usuário ou endereço de e-mail:** wpuser  
**Senha:** `fn6x@N)SIgRT$o!17T`  
![cloud9](img/wordpress-03.png)

19. Ao acessar o console de administração do wordpress, no canto superior esquerdo, clique em **container-technologies**

![cloud9](img/wordpress-04.png)

20. Pronto! Site do wordpress criado com sucesso!

![cloud9](img/wordpress-05.png)

21. Agora vamos definir os limites dos contêineres do nosso serviço, executando o comando abaixo

```shell
docker compose -f compose-limits.yml up -d
```
22. Vamos escalar as réplicas dos contêineres do mysql

```shell
docker compose scale mysql=3
```

Output:
```shell
[+] Running 3/3
 ✔ Container Lab-03-mysql-3  Started            0.1s  
 ✔ Container Lab-03-mysql-1  Started            0.5s   
 ✔ Container Lab-03-mysql-2  Started            0.1s 
```

23. Liste os contêineres dos serviços

```shell
docker compose ps
```

Output:
```output
NAME                IMAGE              COMMAND                  SERVICE     CREATED              STATUS                                  PORTS
Lab-03-mysql-1       mariadb:latest     "docker-entrypoint.s…"   mysql       About a minute ago   Restarting (1) Less than a second ago   
Lab-03-mysql-2       mariadb:latest     "docker-entrypoint.s…"   mysql       About a minute ago   Restarting (1) Less than a second ago   
Lab-03-mysql-3       mariadb:latest     "docker-entrypoint.s…"   mysql       About a minute ago   Up About a minute                       3306/tcp
Lab-03-wordpress-1   wordpress:latest   "docker-entrypoint.s…"   wordpress   2 minutes ago        Up 2 minutes                            0.0.0.0:8080->80/tcp, :::8080->80/tcp
```

24. Setando os limites dos contêineres:

```shell
docker compose -f compose-limits.yml up -d
WARN[0000] /home/ubuntu/environment/container-technologies/Lab-03/compose-limits.yml: `version` is obsolete 
[+] Running 2/3
 ⠧ Network Lab-03_wordpress      Created          0.8s 
 ✔ Container Lab-03-mysql-1      Started          0.3s 
 ✔ Container Lab-03-wordpress-1  Started          0.6s 
```
Conteúdo do **compose-limits.yaml**

```docker-compose
version: "3.8"

services:
  mysql:
    image: mariadb:latest
    command: '--default-authentication-plugin=mysql_native_password'
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '1.5'
          memory: 1024M
        reservations:
          cpus: '1'
          memory: 512M
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: senha1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: UserBlog
      MYSQL_PASSWORD: PwdBlog
    expose:
      - 3306
    volumes:
      - database:/var/lib/mysql
    networks:
      - wordpress

  wordpress:
    image: wordpress:latest
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: UserBlog
      WORDPRESS_DB_PASSWORD: PwdBlog
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
    ports:
      - 8080:80
    networks:
      - wordpress
    depends_on:
      - mysql

volumes:
  database:
  wordpress:

networks:
  wordpress:
    driver: bridge
```

25. Limpando o ambiente

Execute os comandos:
```shell
docker compose -f compose-limits.yml down
```

```shell
docker system prune -a
````
Output: (pressione a tecla: y)

```output
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
```

Laboratório concluído com sucesso!

### Conclusão
Neste laboratório, você aprendeu a configurar e gerenciar serviços utilizando Docker Compose. Foi possível criar um ambiente de desenvolvimento no Cloud9, clonar um repositório e configurar uma stack do WordPress com um banco de dados MySQL. Além disso, você aprendeu a definir limites de recursos para os contêineres, escalar serviços e limpar o ambiente Docker. Esses conhecimentos são essenciais para a orquestração de contêineres e garantem que suas aplicações estejam bem gerenciadas e otimizadas. Parabéns por concluir o laboratório com sucesso!