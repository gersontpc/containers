# Hands-On Kubernetes

Bem-vindos ao laboratório de hands-on de Kubernetes! Durante esta atividade, você irá aprender como criar um cluster Kubernetes utilizando o Elastic Kubernetes Service (EKS) e entender os principais componentes que formam um cluster Kubernetes.

Vamos abordar como criar o deploy de aplicação e quais são componentes necessários, além de escalar o ambiente, verificar os logs dos contêineres e acessar um Pod via SSH.

Ao final deste laboratório, você passará a entender os principais componentes do kubernetes, apto a criar e gerenciar seus próprios clusters Kubernetes na AWS, utilizando as melhores práticas para orquestração de contêineres. Vamos começar!


## O que é o EKS

O Amazon Elastic Kubernetes Service (EKS) é um serviço gerenciado de Kubernetes que simplifica o processo de implantação, operação e manutenção de clusters Kubernetes na Amazon Web Services (AWS).

O EKS fornece um ambiente Kubernetes totalmente gerenciado, que inclui:

Nós do Kubernetes gerenciados pela AWS
Controle de acesso e gerenciamento de identidade
Atualizações e patches do Kubernetes
Monitoramento e alertas
O EKS é uma boa opção para organizações que desejam implantar e gerenciar aplicativos Kubernetes na AWS sem precisar gerenciar a infraestrutura subjacente.

Aqui estão alguns dos benefícios do EKS:

Fácil de configurar e gerenciar: O EKS fornece um ambiente Kubernetes totalmente gerenciado, o que significa que você não precisa se preocupar com a instalação ou manutenção da infraestrutura subjacente.
Segurança: O EKS fornece recursos de segurança integrados, como controle de acesso e gerenciamento de identidade.
Escalabilidade: O EKS é escalável horizontalmente para atender às suas necessidades.
Compatibilidade: O EKS é compatível com o Kubernetes, o que significa que você pode usar as ferramentas e plugins de sua escolha.

Mas existem outras opções de clusters Kubernetes gerenciados em outras clouds publicas, como: Google: Google Kubernetes Engine (GKE) e Azure: Azure Kubernetes Service (AKS).

### Pré-requisitos:
Ter acesso a conta lab no AWS Academy https://awsacademy.instructure.com/.


## Provisionando o EKS

1. Acesse o console da aws e pesquise por EKS e clique em Elastic Kubernetes Service.

![](img/eks-01.png)

2. Ao clicar na página do serviço clique em <b>Create cluster</b>.

![](img/eks-02.png)

Em <b>Configure cluster</b>, selecione a opção <b>Custom configuration</b>.

![](img/eks-03.png)

3. Ao selecionar <b>Custom configuration</b>, perceba que a visualização da criação ela irá mudar (conforme destacado na cor verde, em <b>Custom configuration</b> em Name iremos colocar o nome do nosso cluster, que será `eks-lab`

![](img/eks-04.png)

4. Vá até <b>Auto Mode Compute - <i>New</b></i>, e descelecione a opção <b>Use EKS Auto Mode</b>

![](img/eks-05.png)

5. Em seguida clique em <b>Next</b>

![](img/eks-06.png)

6. Na sessão <b>Specify networking</b>, em <b>VPC</b> deixe a sua <b>VPC Default</b> da conta, em <b>Subnets</b> selecione somente as subnets: <b>us-east-1a</b>, <b>us-east-1b</b> e <b>us-east-1c</b>, conforme o print abaixo.

![](img/eks-07.png)

7. Por último clique em <b>Next</b>

![](img/eks-08.png)

8. Na sessão <b>Configure observability</b>, vá até <b>Control plane logs</b> e habilite os logs para: <b>API Server</b>, <b>Audit</b>, <b>Authenticator</b>, <b>Controller manager</b> e <b>Scheduler</b>.

![](img/eks-09.png)

9. Em seguida clique em <b>Next</b>

![](img/eks-10.png)

10. Na sessão <b>Select add-ons</b>, os <b>Add-ons</b>: <b>kube-proxy</b>, <b>CoreDNS</b>, <b>Amazon VPC CNI</b> e <b>Amazon EKS Pod Identity Agent</b>, já estarão pré-selecionados.

![](img/eks-11.png)

![](img/eks-12.png)

Para que serve cada Add-on ?

> O <b>kube-proxy</b>: Cuida do roteamento interno de tráfego no cluster Kubernetes.  
O <b>CoreDNS</b>: Oferece resolução de nomes, essencial para que os serviços e pods possam se localizar.  
O <b>Amazon VPC CNI</b>: Conecta os pods diretamente à infraestrutura da VPC, garantindo integração nativa com outros serviços AWS.  
O<b> Amazon EKS Pod Identity Webhook</b> (também conhecido como Amazon EKS Pod Identity Agent) é um add-on do Amazon EKS que habilita o uso de IAM Roles for Service Accounts (IRSA). Esse add-on permite que os pods executados em um cluster EKS assumam funções específicas do IAM (IAM roles) usando o mapeamento de identidades, sem precisar configurar credenciais de acesso (chaves de acesso e secret keys) diretamente nos containers.

11. Clique novamente em <b>Next</b>

![](img/eks-13.png)

12. Na sessão <b>Configure selected add-ons settings</b>, clique em <b>Next</b> novamente!
> Esta sessão serve para selecionar as versões dos Add-ons, iremos deizar as versões padrão.

![](img/eks-14.png)

13. Na sessão <b>Review and create</b>, clique em <b>Create</b>.
> Esta sessão serve para revisar todas as configurações que realizamos.

![](img/eks-15.png)

14. Após clicar em <b>Create</b>, você será direcionado para o <b>Cluster</b>, repare que em <b>Status</b> o cluster está em <b>Creating</b>.
> Este passo, demora em torno de 10 minutos.

![](img/eks-16.png)

Enquanto o cluster está sendo criado iremos conhecer o <b>CloudShell</b> da AWS.

### Cloud Shell

Para gerenciar o nosso cluster, iremos utilizar o <b>CloudShell</b> serviço de Terminal Interativo da AWS.

Antes vamos entender o que é o CloudShell!

O AWS Cloud Shell é um ambiente de terminal interativo baseado no navegador que permite que você execute comandos e scripts da AWS CLI. Ele é pré-provisionado com as ferramentas e bibliotecas mais comuns que você precisa para trabalhar com a AWS, incluindo:

- Bash: O shell padrão do Linux
- Python: Uma linguagem de programação popular
- Node.js: Um ambiente de execução JavaScript
- Git: Um sistema de controle de versão
- Kubectl: Ferramenta que se conecta ao API Server para poder gerenciar clusters Kubernetes.

15. No concole da AWS, no canto inferior esquerdo, tem o ícone do Cloud Shell, clique nele!

![](img/eks-17.png)

16. Ao clicar no ícone do CloudShell ele irá provisionar um console para você poder utilizar para gerenciar os serviços da aws através de CLI (Command Line Interface).

![](img/eks-18.png)

> O CloudShell é como se fosse aqueles consoles que existe em data centers, para quem não conhece:

![](img/cd03.png)

Agora que conhece o CloudShell e para que ele serve, iremos utilizar ele até o final desta aula.

![](img/eks-19.png)

Com nosso cluster provisionado, precisamos gerenciar ele, lembra que eu comentei sobre o endpoint do API Server que o cliente (kubectl) se conecta a ele para poder gerenciar o cluster, então agora iremos ver na prática.

### Conhecendo o kubectl

O kubectl é uma ferramenta de linha de comando utilizada para interagir com clusters Kubernetes. Ele permite que os usuários gerenciem aplicativos e recursos em um cluster Kubernetes, como implantações, serviços, pods, configurações, etc.

Com o kubectl, é possível implantar e escalar aplicativos, fazer rolling updates e rollbacks, gerenciar o estado dos pods e serviços, criar e gerenciar objetos personalizados, além de muitas outras tarefas relacionadas ao gerenciamento de um cluster Kubernetes.

O kubectl é uma ferramenta essencial para qualquer pessoa que trabalhe com Kubernetes, como desenvolvedores, administradores de sistemas e operadores de cluster. Ele é usado em um ambiente de terminal para interagir com o cluster Kubernetes e executar operações em qualquer nó ou componente do cluster.

Em resumo, o kubectl é a principal ferramenta de gerenciamento de Kubernetes que permite que você controle e gerencie aplicativos em um cluster Kubernetes.

O <b>CloudShell</b> já vem com o `kubectl` então não precise se preoupar com a instalação, mas caso precise instalar, acesse a documentação: [clique aqui!](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)


### Configurando o Kubectl
17. Agora iremos configurar o autocomplete do `kubectl` e o alias `k` para facilitar na administração do cluster.


Habilitando o auto complete:

```shell
cat <<'EOF' >> ~/.bashrc
# Enable alias and autocomplete
source <(kubectl completion bash)
alias k="kubectl"
complete -o default -F __start_kubectl k
EOF

source ~/.bashrc
```

18. Após configurar o alias, execute o comando abaixo no <b>CloudShell</b>, para configurar o acesso do `kubectl` ao cluster EKS:

``` shell
aws eks update-kubeconfig --region us-east-1 --name eks-lab
```
Output:

```shell
Added new context arn:aws:eks:us-east-1:180359446674:cluster/eks-lab to /home/cloudshell-user/.kube/config
```


Verificando se os componentes do cluster está executando corretamente:

``` shell
kubectl cluster-info
```

```shell
OUTPUT
Kubernetes control plane is running at https://6FE18A9AFD90A9493632AEB363346F9E.gr7.us-east-1.eks.amazonaws.com
CoreDNS is running at https://6FE18A9AFD90A9493632AEB363346F9E.gr7.us-east-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

19. Verificando se o Etcd, Scheduler e o Controller-Manager está funcionando corretamente.

``` shell
kubectl get componentstatuses
```

```shell
OUTPUT:
NAME                 STATUS    MESSAGE   ERROR
etcd-0               Healthy   ok
controller-manager   Healthy   ok
scheduler            Healthy   ok
```

20. Listando os nós do cluster:

``` shell
kubectl get nodes
```

```shell
OUTPUT:
No resources found
```

O que está faltando provisionar?

### Provisionando os Nodes Groups

21. Faltou provisionarmos o nosso <b>Node Group</b>, para provisionar, vá até o seu Cluster <b>eks-lab</b>, clique em <b>Compute</b>:

![](img/eks-20.png)

22. E clique em <b>Add node group</b> para adicionar um novo Node Group

![](img/eks-21.png)

23. Em <b>Name</b>, coloque o nome: `eks-lab-node-group` e em <b>Node IAM role</b>, selecione `LabRole`.

![](img/eks-22.png)

24. Clique em <b>Next</b>

![](img/eks-23.png)

25. Na sessão <b>Set compute and scaling configuration</b>, clique em <b>Next</b>

> Aqui você pode selecionar o tipo de instância, tamanho do disco, a quantidade mínima, desejada e máxima de nodes, além do update.

![](img/eks-23.png)

26. Na sessão <b>Specify networking</b> clique em <b>Next</b>

> Ele selecionará as subnets já setadas no cluster.

![](img/eks-23.png)

27. Na sessão <b>Review and create</b> Revise as configurações e clique em <b>Create</b>.

![](img/eks-24.png)

28. Ao criar o Node Group `eks-lab-node-group`, irá ficar com o status de <b>Creating</b>

> Os nós demora alguns minutos até ficarem prontos.

![](img/eks-25.png)

Após alguns minutos os nós foram criados e estão com o status de <b>Pronto</b> (Ready)

![](img/eks-26.png)

Execute o comando abaixo no CloudShell para visualizar os nós.

```shell
kubectl get nodes
```

29. Listando os componentes do cluster

``` shell
kubectl get all --all-namespaces
```

```shell
OUTPUT:

NAMESPACE     NAME                          READY   STATUS    RESTARTS   AGE
kube-system   pod/aws-node-h4457            2/2     Running   0          174m
kube-system   pod/aws-node-qhdgm            2/2     Running   0          171m
kube-system   pod/coredns-58488c5db-6lbn2   1/1     Running   0          171m
kube-system   pod/coredns-58488c5db-v89qz   1/1     Running   0          172m
kube-system   pod/kube-proxy-x7sqh          1/1     Running   0          171m
kube-system   pod/kube-proxy-zwgdr          1/1     Running   0          174m

NAMESPACE     NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   10.100.0.1    <none>        443/TCP         9h
kube-system   service/kube-dns     ClusterIP   10.100.0.10   <none>        53/UDP,53/TCP   9h

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/aws-node     2         2         2       2            2           <none>          9h
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           <none>          9h

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           9h

NAMESPACE     NAME                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-58488c5db   2         2         2       9h
```

30. Listando os contextos dos clusters configurados:

``` shell
kubectl config get-contexts
```

```shell
OUTPUT:
CURRENT   NAME                                                 CLUSTER                                              AUTHINFO                                             NAMESPACE
          arn:aws:eks:us-east-1:679551794714:cluster/abc       arn:aws:eks:us-east-1:679551794714:cluster/abc       arn:aws:eks:us-east-1:679551794714:cluster/abc
*         arn:aws:eks:us-east-1:679551794714:cluster/eks-lab   arn:aws:eks:us-east-1:679551794714:cluster/eks-lab   arn:aws:eks:us-east-1:679551794714:cluster/eks-lab
```

O kubectl pode ser usado para configurar o contexto de um cluster no arquivo ~/.kube/config. Esse arquivo é usado pelo kubectl para determinar qual cluster, usuário e namespace devem ser usados para interagir com o cluster.

A seguir estão os principais elementos de um contexto configurado no arquivo ~/.kube/config:

`apiVersion`: a versão da API do Kubernetes usada pelo arquivo de configuração.  
`kind`: o tipo do objeto Kubernetes. Para um contexto, o tipo é Context.  
`name`: o nome do contexto.  
`context`: um objeto que define as informações de contexto para um cluster, incluindo cluster, user e namespace.  
`cluster`: um objeto que define as informações do cluster, incluindo o nome do cluster e o endpoint para se conectar ao API server.  
`user`: um objeto que define as informações de autenticação do usuário, como o nome do usuário e a chave de acesso.  
`namespace`: o namespace padrão a ser usado pelo contexto.  

Um exemplo de contexto configurado no arquivo ~/.kube/config pode ser semelhante a este:

```
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWkxsMFErVjNVRGN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1ERXdNRE...kwySmsyd2hKUUppUHRNQ1piUlNKS3hDM2tzWkEKUkV3VExGZG5nR1owCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://SUASUGSHSUAYRSKDKSAUSA.gr7.us-east-1.eks.amazonaws.com
  name: arn:aws:eks:us-east-1:679551794714:cluster/eks-lab
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJZGg4TWlYc2xiUEV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1ERXdOVEkwTWpCYUZ3MHp...RU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://SUASUGSHSUAYRSKDKSAUSA.gr7.us-east-1.eks.amazonaws.com
  name: arn:aws:eks:us-east-1:679551794714:cluster/abc
contexts:
- context:
    cluster: arn:aws:eks:us-east-1:679551794714:cluster/eks-lab
    user: arn:aws:eks:us-east-1:679551794714:cluster/eks-lab
  name: arn:aws:eks:us-east-1:679551794714:cluster/eks-lab
```

### Habilitando o observability do Cluster
31. No cluster clique na aba Add-ons e clique em **Get more add-ons**.

![](img/eks-27.png)

32. Marque o plugin **Amazon CloudWatch Observability**.

![](img/eks-28.png)

33. Depois clique em **Next**.

![](img/eks-29.png)

34. Clique em next novamente.

![](img/eks-30.png)

35. E por último clique em **Create**.

![](img/eks-31.png)

Aguarde a criação...

![](img/eks-32.png)

36. Após alguns minutos o status deve ficar como **Active**.

![](img/eks-33.png)

37. Agora iremos ir até o CloudWatch para visualizar o observability do Cluster.

Em serviços pesquise por **CloudWatch**, após clicar em CloudWatch clique em **Container Insights**.

![](img/eks-34.png)

38. Em [1] **Container Insights** selecione *Service: EKS*, logo após clique em [2] **View in maps**.

![](img/eks-35.png)

O Container Insights para EKS é uma ferramenta poderosa que oferece monitoramento, registro e análise abrangentes para seus containers no Amazon Elastic Kubernetes Service (EKS). Ele fornece uma visão holística da saúde e do desempenho de seus aplicativos em contêiner, permitindo que você:

- Monitoramento Abrangente
 - Solução de Problemas Eficaz
 - Otimização de Desempenho
 - Segurança Aprimorada
 - Integração com Ferramentas Existentes

Benefícios do Container Insights:

**Visibilidade Aprimorada:** Obtenha uma visão completa do seu cluster EKS e dos containers em execução.  
**Solução de Problemas Mais Rápida:** Identifique e resolva problemas de containers de forma mais rápida e eficiente.  
**Desempenho Otimizado:** Otimize o desempenho do seu cluster EKS e dos containers para obter melhores resultados.  
**Segurança Reforçada:** Proteja seus containers e cluster EKS contra ameaças à segurança.  
**Gerenciamento Simplificado:** Simplifique o gerenciamento do seu cluster EKS e dos containers em execução.

![](img/eks-36.png)

### Realizando o deploy de uma aplicação

Agora que já entendemos os principais componentes do Cluster, iremos realizar o deploy de uma app, essa app será basicamente uma imagem do NGINX,  O NGINX é um servidor web de código aberto que pode ser usado para servir arquivos estáticos ou atuar como um proxy reverso para outros servidores web.


39. Primeiro iremos clonar o repositório da aula no Cloud Shell.

```
cd ~/
git clone https://github.com/gersontpc/container-technologies.git
cd ~/container-technologies/Lab-05/02-cluster-eks/
```

Após clonar o repositório, na path /manifests, tem os manifestos para realizarmos o deploy.

Antes iremos preparar o ambiente criando um namespace (isolamento lógico) `nginx` para podermos realizar o deploy da nossa app.

`00-namespace.yaml`

```
---
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
  labels:
    name: nginx
```
Execute o comando abaixo para criar o namespace

```kubernetes
kubectl apply -f manifests/00-namespace.yaml
```

40. Após a criação do namespace, iremos executar o comando para listar todos os namespaces.

``` shell
kubectl get namespace
```

```shell
OUTPUT:
NAME              STATUS   AGE
default           Active   10h
kube-node-lease   Active   10h
kube-public       Active   10h
kube-system       Active   10h
```

Após ter criado o namespace iremos realizar o nosso primeiro o `01-deployment.yaml`, vamos entender os principais componentes deste manifesto.

`manifests/01-deployment.yaml`

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        command: ["/bin/bash"]
        args: ["-c", "echo '<h1>This is the new Kubernetes default page!</h1>' > /usr/share/nginx/html/index.html && exec nginx -g 'daemon off;'"]
        restartPolicy: OnFailure
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "100m"
            memory: "256Mi"
          requests:
            cpu: "50m"
            memory: "128Mi"

```

`apiVersion:` especifica a versão da API que está sendo utilizada, neste caso a apps/v1.  
`kind:` especifica o tipo de objeto que está sendo criado, neste caso um Deployment.  
`metadata:` contém informações de metadados do objeto, como o nome e labels para identificação.  
`replicas:` especifica o número de réplicas que serão criadas para garantir a disponibilidade da aplicação.  
`selector:` especifica o seletor de labels que será usado para identificar os Pods que fazem parte do Deployment.  
`template:` especifica o modelo para criação dos Pods do Deployment.  
`metadata:` contém as informações de metadados do Pod.  
`labels:` especifica as labels para identificação do Pod.  
`spec:` especifica as características do Pod, como os containers que serão executados dentro dele.  
`containers:` especifica o(s) container(s) que serão executados no Pod.  
`name:` especifica o nome do container.  
`image:` especifica a imagem do container que será utilizada.  
`ports:` especifica as portas que serão expostas pelo container. Neste caso, a porta 80 foi exposta.  

41. Após entender cada item do nosso manifesto, vamos utilizar ele para realizar o deploy do nosso app.


``` shell
kubectl create -f manifests/01-deployment.yaml
```

```shell
OUTPUT:
deployment.apps/nginx created
```

42. Após aplicado, iremos listar a nossa app. Repare que os pods já estão em status de Running.

``` shell
kubectl get pods -n nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-57787bb6df-2sgx7   1/1     Running   0          37s
nginx-57787bb6df-k27wr   1/1     Running   0          37s
nginx-57787bb6df-lvrp4   1/1     Running   0          37s
```

43. Trazendo informação mais detalhada dos pods.

``` shell
kubectl get pods -n nginx -o wide
```

```shell
OUTPUT:
NAME                     READY   STATUS    RESTARTS   AGE    IP              NODE                            NOMINATED NODE   READINESS GATES
nginx-57787bb6df-2sgx7   1/1     Running   0          2m1s   172.31.6.187    ip-172-31-12-186.ec2.internal   <none>           <none>
nginx-57787bb6df-k27wr   1/1     Running   0          2m1s   172.31.3.92     ip-172-31-12-186.ec2.internal   <none>           <none>
nginx-57787bb6df-lvrp4   1/1     Running   0          2m1s   172.31.89.245   ip-172-31-93-147.ec2.internal   <none>           <none>
```

44. Conhecendo o POD da nossa app, com o comando `kubectl describe pods <pod-name>`

``` shell
kubectl describe pods nginx-57787bb6df-2sgx7 -n nginx
```

```shell
OUTPUT:
Name:             nginx-57787bb6df-2sgx7
Namespace:        nginx
Priority:         0
Service Account:  default
Node:             ip-172-31-12-186.ec2.internal/172.31.12.186
Start Time:       Wed, 01 Nov 2023 10:34:57 +0000
Labels:           app=nginx
                  pod-template-hash=57787bb6df
Annotations:      <none>
Status:           Running
IP:               172.31.6.187
IPs:
  IP:           172.31.6.187
Controlled By:  ReplicaSet/nginx-57787bb6df
Containers:
  nginx:
    Container ID:  containerd://c35a0fa58303cdb880aa93533a72eedfed286f468cceb71e7e4d4cc3338caa4a
    Image:         nginx:latest
    Image ID:      docker.io/library/nginx@sha256:775b948ed18e931cfe48c4664387162764dcf286303bbe02bd72703f72f17f02
    Port:          80/TCP
    Host Port:     0/TCP
    Command:
      /bin/bash
    Args:
      -c
      echo '<h1>Hello K8s World</h1>' > /usr/share/nginx/html/index.html && exec nginx -g 'daemon off;'
    State:          Running
      Started:      Wed, 01 Nov 2023 10:35:06 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  256Mi
    Requests:
      cpu:        50m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lxp86 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-lxp86:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m27s  default-scheduler  Successfully assigned nginx/nginx-57787bb6df-2sgx7 to ip-172-31-12-186.ec2.internal
  Normal  Pulling    3m23s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     3m19s  kubelet            Successfully pulled image "nginx:latest" in 4.843s (4.843s including waiting)
  Normal  Created    3m19s  kubelet            Created container nginx
  Normal  Started    3m18s  kubelet            Started container nginx
```

45. Conhecendo o nosso deploy, utilizando o comando `kubectl describe deploy <deploy-name>`, mas antes iremos pegar o nome do nosso deploy.  
Primeiro vamos realizar o `kubectl get deploy` para listar todos os deploys de todos os namespaces.


``` shell
kubectl get deploy nginx -n nginx
```

```shell
OUTPUT:
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           115m
```

40. Descrevendo o deploy `nginx`.

``` shell
kubectl describe deploy nginx -n nginx
```

```shell
OUTPUT:
Name:                   nginx
Namespace:              nginx
CreationTimestamp:      Wed, 01 Nov 2023 10:34:57 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 100% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:      nginx:latest
    Port:       80/TCP
    Host Port:  0/TCP
    Command:
      /bin/bash
    Args:
      -c
      echo '<h1>Hello K8s World</h1>' > /usr/share/nginx/html/index.html && exec nginx -g 'daemon off;'
    Limits:
      cpu:     100m
      memory:  256Mi
    Requests:
      cpu:        50m
      memory:     128Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-57787bb6df (3/3 replicas created)
Events:          <none>
```

### Acessando a App

Para acessar a app, precisamos expor o serviço em nosso cluster apontando para os nossos pods.

Criando a service para poder acessar a app, o manifesto está na path `manifests/02-service-nginx.yaml`.
O conteúdo do manifesto é:

`manifests/02-service-nginx.yaml`


```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
````

Este manifesto descreve um serviço do Kubernetes com as seguintes configurações:

`apiVersion:` a versão da API do Kubernetes usada para criar o recurso. Neste caso, é a versão "v1". 
`kind:` o tipo do recurso Kubernetes que está sendo criado. Neste caso, é um "Service".  
`metadata:` informações sobre o objeto sendo criado, incluindo um nome descritivo e rótulos que permitem que o recurso seja pesquisado e gerenciado com facilidade.  
`spec:` as especificações do serviço, incluindo sua configuração de rede e informações de porta.  
`selector:` uma seleção baseada em rótulos que identifica os pods a serem expostos pelo serviço. Neste caso, o serviço expõe todos os pods com o rótulo "app: nginx".  
`type:` o tipo de serviço a ser criado. Neste caso, é um LoadBalancer, que permite fazer o balanceamento de carga entre os ips dos nós  
`ports:` as portas expostas pelo serviço. Neste caso, uma única porta é exposta com o nome "http" e mapeada para a porta 80 nos pods correspondentes à seleção de rótulo do serviço.


41. Após entender cada item do nosso manifesto, vamos utilizar ele para realizar o deploy da nossa service.

``` shell
kubectl apply -f manifests/02-service-nginx.yaml
```

```shell
OUTPUT:
service/nginx-service created
```

46. Antes de acessar o app pelo browser, vamos liberar o security group dos Node Groups para acessar a app, vá até o cluster EKS, aba <b>Networking</b> e em <b>Cluster security group</b>, clique no id do security group [sg-00e6a5104f3b24d2a].

![](img/eks-37.png)

47. Na aba <b>Inbound rules</b> clique em <b>Edit inbound rules</b>

![](img/eks-38.png)

48. [1] Clique em <b>Add rule</b>, [2] selecione a regra <b>HTTP</b>, libere para qualquer origem <b>0.0.0.0/0</b>, [3] clique em <b>Save rules</b>, para salvar a regra.

![](img/eks-39.png)

49. Acessado a nossa "app" via Load Balancer.

``` shell
kubectl get service -n nginx
```

```shell
OUTPUT:
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP                                                              PORT(S)        AGE
nginx   LoadBalancer   10.100.84.30   a0d10f8234df541fc807413cb72fe8dc-844940680.us-east-1.elb.amazonaws.com   80:32469/TCP   14s
```

Repare que a nossa service `nginx` foi criada com sucesso.

50. Agora vamos acessar o app através do navegador, digite a url do Load Balancer:

```
http://a1b5767bf329144d69671d9377adcdc8-228760430.us-east-1.elb.amazonaws.com/
```
![](img/eks-40.png)

Sucesso! conseguimos acessar a nossa app.

### Escalando os PODs

51. Para escalar os pods iremos executar o comando `kubectl scale deployment`, atualmente temos 2 répicas e iremos aumentar para 8 réplicas.

``` shell
kubectl scale deployment nginx --replicas=8 -n nginx
```

```shell
OUTPUT:
deployment.apps/nginx scaled
```

52. Checando se foi escalado corretamente.

``` shell
kubectl get pods -n nginx
```

```shell
OUTPUT:
NAME                     READY   STATUS              RESTARTS   AGE
nginx-57787bb6df-2sgx7   1/1     Running             0          128m
nginx-57787bb6df-dk7wt   0/1     ContainerCreating   0          3s
nginx-57787bb6df-k27wr   1/1     Running             0          128m
nginx-57787bb6df-lvrp4   1/1     Running             0          128m
nginx-57787bb6df-m5kch   0/1     ContainerCreating   0          3s
nginx-57787bb6df-r2sgf   0/1     ContainerCreating   0          3s
nginx-57787bb6df-r4qzq   0/1     ContainerCreating   0          3s
nginx-57787bb6df-sn64c   0/1     ContainerCreating   0          3s
```

55. Repare que os containers foram escalados para 8 réplicas.

``` shell
kubectl get pods -n nginx
```

```shell
OUTPUT:
NAME                     READY   STATUS    RESTARTS   AGE
nginx-57787bb6df-2sgx7   1/1     Running   0          3h29m
nginx-57787bb6df-dk7wt   1/1     Running   0          80m
nginx-57787bb6df-k27wr   1/1     Running   0          3h29m
nginx-57787bb6df-lvrp4   1/1     Running   0          3h29m
nginx-57787bb6df-m5kch   1/1     Running   0          80m
nginx-57787bb6df-r2sgf   1/1     Running   0          80m
nginx-57787bb6df-r4qzq   1/1     Running   0          80m
nginx-57787bb6df-sn64c   1/1     Running   0          80m
```

Pronto os PODs foram escalados com sucesso!

53. Vamos voltar para duas réplicas.

``` shell
kubectl scale deployment nginx --replicas=2 -n nginx
```

```shell
OUTPUT:
deployment.apps/nginx scaled
```

### Realizando rollback de versão

54. Para realizar o rollback da versão iremos aplicar o manifesto `manifests/deployment-v1.yaml`

``` shell
kubectl apply -f manifests/03-deployment-v1.yaml
```

```shell
OUTPUT:
deployment.apps/nginx-deployment configured
```

55. Execute o curl para verificar se alterou a página do nosso webserver.

``` shell
curl http://a0d10f8234df541fc807413cb72fe8dc-844940680.us-east-1.elb.amazonaws.com/
```

```shell
OUTPUT:
<h1>Deploy V1!</h1>
```

Ou acesse pelo browser:  
![](img/deploy-v1.png)

56. Após realizar o deploy, vamos realizar o deploy do `manifests/deployment-v2.yaml`

``` shell
kubectl apply -f manifests/04-deployment-v2.yaml
```

```shell
OUTPUT:
deployment.apps/nginx-deployment configured
```

57. Execute o curl para verificar se alterou a versão.

``` shell
curl http://a0d10f8234df541fc807413cb72fe8dc-844940680.us-east-1.elb.amazonaws.com/
```

```shell
OUTPUT:
<h1>Deploy V2!</h1>
```
Ou acesse pelo browser:  
![](img/deploy-v2.png)

58. Repare que os pods estão recém criados:
``` shell
kubectl get pods -l app=nginx -n nginx
```

```shell
OUTPUT:
NAME                    READY   STATUS    RESTARTS   AGE
nginx-5fd47c7f7-28h4z   1/1     Running   0          10s
nginx-5fd47c7f7-gg8qj   1/1     Running   0          10s
nginx-5fd47c7f7-mqfm5   1/1     Running   0          10s
```

59. Agora iremos realizar o Rollback para a versão 1 do nosso deployment. O Deployment cria uma revisão a cada alteração no deploy, para listar essas versões iremos utilizar o `kubectl rollout history`:

``` shell
kubectl rollout history deploy nginx -n nginx
```

```shell
OUTPUT:
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

60. Agora vamos realizar o rollback para a versão 2 da nossa app.

``` shell
kubectl rollout undo deployment/nginx --to-revision=1 -n nginx
```

```shell
deployment.apps/nginx rolled back
```

61. Vamos checar se ele alterou para a versão que fizemos a nossa alteração:

    Realize o curl para verificar se estamos na Versão 2 da app, lembrando que na linha do tempo a versão 2 do histório de deploy, deve retornar a página com a frase <b>Deploy V1!</b>.

``` shell
curl http://a0d10f8234df541fc807413cb72fe8dc-844940680.us-east-1.elb.amazonaws.com/
```

```shell
OUTPUT:
<h1>Deploy V1!</h1>
```
Ou acesse pelo browser:  
![](img/deploy-v1.png)

Pronto, agora você acabou de aprender a fazer rollback para uma versão específica da sua app.


### Finalizando o Lab

Agora é hora de limpar a casa! :)

Vamos deletar os objetos que criamos dentro do nosso cluster Kubernetes

62. Deletando a Service:

``` shell
kubectl delete -f manifests/02-service-nginx.yaml
```

```shell
OUTPUT:
service "nginx" deleted
```

63. Deletendo o Deployment:

``` shell
kubectl delete -f manifests/01-deployment.yaml
```

```shell
OUTPUT:
deployment.apps "nginx" delete
```

Excluindo o <b>Node Group</b> do Cluster EKS.

64. Acesse o serviço [EKS](https://us-east-1.console.aws.amazon.com/eks/home?region=us-east-1#/clusters)

![](img/eks-01.png)

65. Clique no cluster do lab `eks-lab`

![](img/eks-41.png)

66. Clique na aba <b>Compute</b>

![](img/eks-42.png)

67. Desça até <b>Node groups</b>, selecione o `eks-lab-node-group` e clique em <b>Delete</b>

![](img/eks-43.png)

68. Por questão de segurança você precisa colocar o nome do node group para ter certeza que precisa deletar ele (isso evita a deleção acidental), depois clique em <b>Delete</b>!

![](img/eks-44.png)

69. Agora basta esperar alguns minutos para realizar a deleção  

![](img/eks-45.png)

70. Pronto, após a deleção do node group, iremos realizar a deletar o Cluster, no cluster clique em <b>Delete cluster</b>.

![](img/eks-46.png)

71. Para deletar coloque o nome do cluster <b>eks-lab</b> para confirmar a deleção, logo em seguida clique em <b>Delete</b>

![](img/eks-47.png)

72. Após confirmar a deleção, o cluster será deletado em alguns minutos.

![](img/eks-48.png)


### Conclusão
Pronto, você acaba de concluir o laboratório de hands-on de EKS!

Durante este laboratório, você aprendeu como criar um cluster EKS e alguns dos principais componentes que compõem um cluster Kubernetes. Você também aprendeu como criar um namespace, deployment, service do tipo load balancer e realizar o scaling e rollback de versão da app.

Este conhecimento é fundamental para quem deseja entender e trabalhar com Kubernetes, uma das plataformas mais utilizadas no mundo para orquestração de containers. Esperamos que este laboratório tenha sido útil para você e que agora você se sinta mais confiante para trabalhar com Kubernetes em seu ambiente. Parabéns novamente e continue praticando!
