# Lab 5: Fazendo deploy de uma aplicação utilizando IBM Toolchain


Nesse lab, você aprenderá como fazer o deploy da mesma aplicação guestbook que usamos
nos labs anteriores, utilizando a IBM Toolchain. 

Clone o repositório:

```
$ git clone https://github.com/IBM/guestbook.git
```


# 1. Crie sua toolchain

Uma toolchain é um grupo de ferramentas integradas para desenvolvimento, monitoração e mais. Você pode criar sua toolchain do zero ou se basear em um template.

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
  type: NodePort
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
