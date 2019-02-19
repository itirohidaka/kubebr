# Lab 3: Escale e atualize aplicações nativamente, construindo aplicações multi-tier.

Nesse lab, você aprenderá como fazer o deploy da mesma aplicação guestbook que fizemos
no lab anterior, de qualquer maneira, ao invés de usar as funções do `kubectl`
estaremos fazendo o depoy da aplicação usando arquivos de configuração.
The configuration file mechanism allows you to have more
O modo de arquivos de configuração permite que você tenha um controle maior 
sobre os recursos que estão sendo criados dentro do seu cluster Kubernetes.

Antes de trabalharmos com a aplicação, nós precisamos clonar o repositório do github:

```
$ git clone https://github.com/IBM/guestbook.git
```

Esse repositório contém várias versões da aplicação guestbook, 
assim como os arquivos de configuração que vamos utilizar para fazer o deploy das partes da aplicação.
Mude o diretório executando o comando `cd guestbook`. Você encontrará os arquivos 
de configuração para esse exercício no diretório `v1`.

# 1. Escale as aplicações nativamente

O Kubernetes pode fazer o deploy em um pod individual para rodar a aplicação 
mas quando você precisa escalar para lidar com uma grande quantidade de requests o  `Deployment` é o
que você vai utilizar.

Um deployment gerencia uma coleção de pods semelhantes. Quando você definir um número específico de replicas, 
o Kubernetes Deployment Controller vai manter esse número de replicas o tempo todo. 

Cada objeto do Kubernetes que criamos deve fornecer dois campos de objeto aninhados 
que controlam a configuração do objeto: o objeto  `spec` e o objeto
`status`. O objeto `spec` define o estado desejado enquanto o objeto `status` contem 
toda informação provida do sistema do Kubenetes sobre o atual estado do recurso. 
Como descrito anteriormente, Kubernetes tentará sempre manter o estado 
atual igual ao estado desejado do sistema.

Para o objeto que criamos, nós precisamos fornecer a `apiVersion` que você está usando 
para criar o objeto, o `kind` do objeto que estamos criando e o `metadata`
sobre o objeto, assim como `name`, grupo de `labels` e opcionalmente `namespace`
que esse objeto deveria pertencer.

Considerando a configuração do seguinte deployment para a aplicação guestbook:

**guestbook-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - name: guestbook
        image: ibmcom/guestbook:v1
        ports:
        - name: http-server
          containerPort: 3000
```

O arquivo de configuração acima cria um objeto deployment chamado 'guestbook'
com um pod contendo um único container rodando a imagem
`ibmcom/guestbook:v1`.  Esse arquivo também define a quantidade de replicas 
igual a 3 e o Kubernetes tenta sempre manter pelo menos 3 pods ativos rodando 
o tempo todo.

- Criar deployment do guestbook

  Para criar um Deployment usando esse arquivo de configuração, 
execute o seguinte comando:

   ``` console
   $ kubectl create -f guestbook-deployment.yaml
   deployment "guestbook" created
   ```

- Liste os pods com label app=guestbook

  Nós podemos listar os pods criados listando todos os pods que possuem 
  uma label com valor igual a "guestbook". Isso corresponde 
  as labels definidas acima no arquivo yaml na seção 
  `spec.template.metadata.labels` section.

   ```console 
   $ kubectl get pods -l app=guestbook
   ```

 Quando você muda o número de replicas em uma configuração, o Kubernetes vai tentar
 adicionar ou remover pods do sistema correspondente a sua requisição. Para fazer 
 essas alterações, execute o seguinte comando:

   ```console
   $ kubectl edit deployment guestbook
   ```

Isso fará com que ele busque no servidor do Kubernetes a última configuração 
do Deployment e carregará em um editor para você. Você vai perceber que existem 
mais campos nessa versão do que no arquivo yaml utilizado. Nesse arquivo contem 
todas as propriedades sobre o deployment que o Kubernetes possui conhecimento, 
não apenas aqueles que nós escolhemos especificar quando criamos. Note também que 
agora contem seção de status mencionada anteriormente.

Você pode também editar o arquivo de deployment que nós usamos para criar o 
Deployment para fazer alterações. Para isso, você deve usar o seguinte comando 
para fazer uma mudança efetiva quando você editar o deployment locamente.

   ```console
   $ kubectl apply -f guestbook-deployment.yaml
   ```

Isso solicitará ao kubernetes para diferenciar nosso arquivo yaml do 
estado atual do Deployment e aplicar apenas essas mudanças.

Nós podemos definir agora um objeto Service para expor nosso deployment
para clients externos.

**guestbook-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  ports:
  - port: 3000
    targetPort: http-server
  selector:
    app: guestbook
  type: LoadBalancer
```

A configuração acima cria um recurso Service chamado guestbook. Um Service
pode ser usado para criar um caminho de rede para o tráfego de entrada para 
a aplicação que está rodando. Nesse caso, nós estamos preparando um rota da 
porta 3000 no cluster para a porta “http-server” do nosso app, que é a porta 
3000 de acordo com a especificação do Deployment.


- Agora iremos criar o guestbook service usando um comando parecido ao
  que usamos quando criamos o Deployment:

  ` $ kubectl create -f guestbook-service.yaml `

- Teste o app guestbook usando o browser que preferir com a URL
  `<seu-cluster-ip>:<node-port>`

  Lembre-se, para pegar a `nodeport` e o `public-ip` use:

  `$ kubectl describe service guestbook`
  e
  `$ ibmcloud cs workers <nome-do-cluster>`

# 2. Conecte-se a um serviço back-end.

Se você olhar o código fonte do guestbook, no diretório `guestbook/v1/guestbook`,
você irá notar que foi escrito para suportar uma variedade de banco de dados.
Por padrão irá manter o log do guestbook inteiro na memória.
Não tem problema para fins de teste, mas assim que você começar a entrar em ambientes mais "reais"
onde você escala a sua aplicação, esse modelo não vai atender porque
com base na instância da aplicação para o qual o usuário é roteado, ele verá resultados muito diferentes

Para resolvermos isso, nós precisamos ter todas as instâncias do nosso app compartilhando o mesmo banco
de dados - nesse caso nós vamos usar um banco de dados redis que provisionamos no nosso cluster.
Essa instância do redis vai ser definida de uma maneira parecida com a do guestbook.

**redis-master-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
      - name: redis-master
        image: redis:2.8.23
        ports:
        - name: redis-server
          containerPort: 6379
```

Esse yaml cria um banco de dados redis em um Deployment chamado 'redis-master'.
Isso irá criar um instância única, com as replcias definidas como 1 e as instâncias da aplicação guestbook
se conectarão a ele para persistir os dados, além de ler os dados persistentes de volta.
A imagem que está rodando no container é a 'redis:2.8.23' e expoe a porta padrão do redis 6379.

- Crie um redis Deployment, como fizemos para o guestbook:

    ```console
    $ kubectl create -f redis-master-deployment.yaml
    ```

- Verifique se o pod do redis server está "running":

    ```console
    $ kubectl get pods -lapp=redis,role=master
    NAME                 READY     STATUS    RESTARTS   AGE
    redis-master-q9zg7   1/1       Running   0          2d
    ```

- Vamos testar o redis standalone:

    ` $ kubectl exec -it redis-master-q9zg7 redis-cli `

    O comando kubectl exec irá iniciar um processo secundário no container especificado.
    Neste caso, estamos pedindo que o comando "redis-cli" seja executado 
    no container chamado "redis-master-q9zg7".Quando esse processo se encerra
    o comando "kubectl exec" também se encerrará mas os outros processos
    no container não serão afetados.

    Uma vez no container nós podemos usar o comando "redis-cli" para ter certeza que o
    bando de dados redis está executando normalmente, ou para configurá-lo se precisar.

    ```console
    redis-cli> ping
    PONG
    redis-cli> exit
    ```

Agora precisamos expor o Deployment `redis-master` como um serviço para que a
aplicação guestbook possa se conectar através do DNS lookup. 

**redis-master-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: master
```

Isso cria um objeto Service chamado 'redis-master' e configura porta de destino
6379 nos pods selecionados pelos seletores "app=redis" e "role=master".

- Cria o serviço para acessar o redis master:

    ``` $ kubectl create -f redis-master-service.yaml ```

- Reinicie o guestbook para que ele encontre o serviço redis para usar como banco de dados:

    ```console
    $ kubectl delete deploy guestbook 
    $ kubectl create -f guestbook-deployment.yaml
    ```

- Teste o app guestbook utilizando qualquer browser que preferir com a URL:
  `<your-cluster-ip>:<node-port>`
  
Você pode verificar que se você abrir vários browsers e atualizar as páginas
para acessar as diferentes cópias do guestbook, todas elas terão um estado consistente.
Todas as instâncias gravam no mesmo armazenamento persistente de backup e todas as instâncias
leem nesse armazenamento para exibir as entradas do guestbook que foram armazenadas.


Nós temos nossa simples aplicação 3-tier rodando, mas precisamos escalar a aplicação
caso o tráfego aumente. Nosso principal problema é que temos apenas 
um servidor de banco de dados para processar cada request que chega através do guestbook.
Uma solução simples seria separar os request de read e write de uma forma que eles iriam
para diferentes banco de dados que sejam replicados para obter consistência de dados.

![rw_to_master](../images/Master.png)

Crie um deployment chamado 'redis-slave' para que possamos conversar com o banco de dados redis
para gerenciar a leitura de dados. Para escalarmos o banco de dados, nós usamos um padrão onde
podemos escalar a leitura utilizando deployment redis slave que pode rodar diversas
instâncias para leitura. O deployment redis slave é configurado para rodar duas replicas.

![w_to_master-r_to_slave](../images/Master-Slave.png)

**redis-slave-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
      - name: redis-slave
        image: kubernetes/redis-slave:v2
        ports:
        - name: redis-server
          containerPort: 6379
```

- Crie o pod rodando deployment redis slave.
 ``` $ kubectl create -f redis-slave-deployment.yaml ```

 - Verifique se todas as replicas slave estão rodando
 ```console
$ kubectl get pods -lapp=redis,role=slave
NAME                READY     STATUS    RESTARTS   AGE
redis-slave-kd7vx   1/1       Running   0          2d
redis-slave-wwcxw   1/1       Running   0          2d
 ```

- Verifique um dos pods para checar o banco de dados e ver se 
  tudo está certo:

 ```console
$ kubectl exec -it redis-slave-kd7vx  redis-cli
127.0.0.1:6379> keys *
1) "guestbook"
127.0.0.1:6379> lrange guestbook 0 10
1) "hello world"
2) "welcome to the Kube workshop"
127.0.0.1:6379> exit
```

Faça o deploy do serviço redis slave para que possamos acessá-lo pelo nome DNS. Quando feito o redeploy,
a aplicação vai mandar operações de leitura para os pods do`redis-slave`
enquanto operações de gravação vão para os pods do `redis-master`.
**redis-slave-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: slave
```

- Crie o serviço para acessar o redis slaves.
    ``` $ kubectl create -f redis-slave-service.yaml ```

- Reinicie o guestbook para que ele encontre o serviço slave para ler.
    ```console
    $ kubectl delete deploy guestbook
    $ kubectl create -f guestbook-deployment.yaml
    ```
    
- Teste a aplicação guestbook utilizando qualquer browser que preferir usando a URL `<your-cluster-ip>:<node-port>`.

Este é o fim do lab. Agora vamos limpar nosso ambiente:

```console
$ kubectl delete -f guestbook-deployment.yaml
$ kubectl delete -f guestbook-service.yaml
$ kubectl delete -f redis-slave-service.yaml
$ kubectl delete -f redis-slave-deployment.yaml 
$ kubectl delete -f redis-master-service.yaml 
$ kubectl delete -f redis-master-deployment.yaml
```
