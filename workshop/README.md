# IBM Cloud Kubernetes Service Lab

<img src="https://kubernetes.io/images/favicon.png" width="200">

# Uma introdução à containers

Ei, você está procurando um curso 101 sobre containers? Confira nosso [Docker Essentials](https://developer.ibm.com/courses/all/docker-essentials-extend-your-apps-with-containers/).

Containers permitem que você rode aplicações isoladas com segurança with quotas on system resources. Containers começaram como uma ferramenta individual entregue com kernel linux. Docker foi lançado tornando containers fácil de se usar e os desenvolvedores rapidamente aderiram essa ideia. Containers também despertaram interesses em arquiteturas de microsserviços, uma metodologia para desenvolvimento de aplicações aonde aplicações complexas são divididas em partes essas partes trabalham juntas.

Assista esse [video](https://www.youtube.com/watch?v=wlBhtc31I8c) para saber mais sobre usos de containers em produção.

# Objetivos

Esse lab é uma introdução ao uso de containers no IBM Cloud Kubernetes Service. No final do curso, você alcançará os seguintes objetivos:
* Entender os principais conceitos de Kubernetes
* Construir uma imagem Docker e fazer o deploy de uma aplicação na IBM Cloud Kubernetes Service 
* Controlar deployments de aplicações, enquato minimiza seu tempo com gerenciamento de infraestrutura
* Adicionar serviços de IA para ampliar sua aplicação 
* Proteger e monitorar seu cluster e sua aplicação

# Pré-requisitos 
* Uma conta Pay-As-You-Go ou de assinatura [IBM Cloud account](https://console.bluemix.net/registration/)

# Máquinas Virtuais

Antes dos containers, a maior parte da infraestrutura não rodava em bare metal, mas sobre hypervisors gerenciando vários sistemas operacionais virtualizados (OSes). Esse arranjo permite o isolamento de aplicações uma das outras em um nivel maior do que o provido pelo sistema operacional. Esses sistemas operacionais virtualizados veem o que parece pra eles o próprio hardware exclusivo. Contudo, isso também significa que cada sistema operacional virtualizado replicam um SO inteiro, ocupando espaço no disco.

# Containers

Containers fornecem um isolamento parecido com as VMs, exceto pelo SO e o nível do processo. Cada container é um processo ou grupo de processos que rodam isolados. Containers típicos rodam apenas um unico processo, por não precisarem dos serviços padrão do sistema. O que eles geralmente precisam fazer pode ser fornecido pelas chamadas do sistema para o kernel do sistema operacional base.

O isolamento no linux é fornecido pela feature chamada 'namespaces'. Cada diferente tipo de isolamento (usuário IE, cgroups) é fornecido por um diferente namepsace.

Essa é uma lista de alguns namespaces que são normalmente usados e visíveis ao usuário:

* PID - process IDs
* USER - user and group IDs
* UTS - hostname and domain name
* NS - mount points
* NET - network devices, stacks, and ports
* CGROUPS - control limits and monitoring of resources

# VM vs container

Aplicações tradicionais rodam em um hardware nativo. Uma única aplicação geralmente não utiliza todos os recursos de uma única máquina.  Nós tentamos rodar múltiplas aplicações em uma única máquina para evitar deixar recursos ociosos. Nós poderíamos rodas várias cópias de uma mesma aplicação, mas para fornecer isolamento nós usamos VMs que rodam várias instâncias (VMs) em um mesmo hardware. Essas VMs possuem pilhas de sistemas operacionais que as tornam relativamente grandes e ineficientes  devido a duplicação tanto do runtime quanto do disco.

![Containers versus VMs](images/VMvsContainer.png)

Containers permitem que você compartilhe um SO host. Isso reduz as duplicações e ainda fornece o isolamento. Containers também permitem que você compartilhe um SO host. Containers também permitem que você reduza arquivos desnecessários assim como bibliotecas de sistemas e binários para economizar espaço e reduzir a superfície de ataque. Se o SSHD ou a LIBC não estiverem instalados, eles não poderão ser utilizados.

# Prepare-se

Antes de entrarmos em Kubernetes, você precisará provisionar um cluster para você conteinerizar sua aplicação. Então você não terá que esperar ficar pronto para os labs subsequentes. 

1. Você deve instalar as CLIs pelo https://console.ng.bluemix.net/docs/containers/cs_cli_install.html. Se você ainda não possuir essas CLIs e a CLI do Kubernetes, faça o [lab 0](Lab0) antes de iniciar este curso.
2. Se você ainda não possuir um cluster, provisione. Isso pode levar alguns minutos, então comece: `ibmcloud cs cluster-create --name <name-of-cluster>`
3. Depois da criação, antes de usar o cluster, certifique-se que o provsionamento esteja completo e pronto para uso. Rode o seguinte comando: `ibmcloud cs clusters` e verifique se o estado do seu cluster consta como "deployed".  
4. E então rode `ibmcloud cs workers <name-of-cluster>` e verifique se todos os workers nodes estão em state "normal" com os status "Ready".

# Kubernetes e containers: um overview

Vamos discutir sobre orquestração Kubernetes para containers antes de construirmos aplicações nele. Nós precisamos entender os seguintes fatos sobre isso:

* O que extamente é o Kubernetes?
* Como o Kubernetes foi cirado?
* Arquitetura do Kubernetes
* Modelo de recurso do Kubenetes
* Kubernetes na IBM
* Let's get started

# O que é Kubernetes?

Agora que nós sabemos o que são containers, vamos definir o que é o Kubernetes. Kubernetes é um orquestrador de containers para provisionar, gerenciar e escalar aplicações. Em outras palavras, Kubernetes permite que você gerencie o ciclo de vida de aplicações conteinerizadas dentro de um cluster de nodes (que é uma coleção de máquina workers, por exemplo, VMs, máquinas físicas etc.).


Sua aplicação pode precisar de outros recursos como volumes, redes e secrets que vão te ajudar a fazer coisas como conexão com banco de dados, conversar com backend com firewall e secure keys. Kubernetes ajuda você a adicionar esses recursos na sua aplicação. Recursos de infraestrutura necessários da aplicação são gerenciados delcarativamente.

**Observação:** Mesos e Swarm são outras tecnologias de orquestração.  

O paradigma chave é o modo declarativo. O usuário fornece o estado desejado enquanto o Kubenetes fará tudo possível para atender. Se você precisar de 5 instâncias, você não inicia 5 instâncias separadas por conta própria mas diz para o kubernetes que você precisa de 5 instâncias e o kubernetes vai atender esse estado automaticamente. Simplesmente nesse momento você precisa saber que você declara o estado que você quer e o Kubernetes fará isso acontecer. Se alguma coisa der errado com uma das instâncias e a instância cair, Kubernetes ainda saberá o estado desejado e criará novas instâncias no node disponível.

**Fun fact** Kubernetes pode ser chamado por muitos nomes. As vezes é encurtado como _k8s_ (perdendo 8 letras), ou carinhosamente chamado de _kube_. A palavra vem na Grécia antiga que significa "Helmsman" (timoneiro). O helmsman é a pessoa responsável por dirigir o barco. Esperamos que você tenha percebido a analogia entra dirigir um barco e as decisões feitas para orquestrar um container em um cluster.

# Como o Kubernetes foi criado?

Google wanted to open source their knowledge of creating and running the internal tools Borg & Omega. It adopted Open Governance for Kubernetes by starting the Cloud Native Computing Foundation (CNCF) and giving Kubernetes to that foundation, therefore making it less influenced by Google directly. Many companies such as RedHat, Microsoft, IBM and Amazon quickly joined the foundation.

Main entry point for the kubernetes project is at [http://kubernetes.io](http://kubernetes.io) and the source code can be found at [https://github.com/kubernetes](https://github.com/kubernetes).

# Arquitetura do Kubenetes

Em sua essência, Kubernetes é um data store (etcd). O modo declarativo é armazenado dentro do data store como objeto, que significa que quando você diz que quer 5 instâncias de um container, aquele request é armazenado no data store. Essa mudança de informação é vigiada e delegada para os controllers que tomam a ação. Os Controllers reagem então ao modelo e tentam tomar uma ação para atingir ao estado desejado. O poder do Kubernetes está no modelo simplista.

Como monstrado, API server é um simples manipulador de HTTP server que realiza operações como criação/leitura/atualização/exlusão(CRUD) no data store. Então o controller pega a mudança que você deseja e realiza aquela mudança. Controllers são responsáveis por instanciar os recursos atuais representados por qualquer recurso do Kubernetes. Esses recursos atuais são o que a sua aplicação precisa para permitir que execute perfeitamente.

![architecture diagram](images/kubernetes_arch.png)

# Modelo de recursos do Kubernetes

A infraestrutura do kubernetes define um recurso para cada  defines a resource for every propósito. Cada recurso é monitorado e processado pelo controller. Quando você define sua aplicação, ela contem uma coleção desses recursos. Esta coleção poderá então ser lida pelos controlles para construir suas aplicações build your applications actual backing instances. Some of resources that you may work with are listed below for your reference, for a full list you should go to [https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/). In this class we will only use a few of them, like Pod, Deployment, etc.

* Config Maps holds configuration data for pods to consume.
* Daemon Sets ensure that each node in the cluster runs this Pod
* Deployments defines a desired state of a deployment object
* Events provides lifecycle events on Pods and other deployment objects
* Endpoints allows a inbound connections to reach the cluster services
* Ingress is a collection of rules that allow inbound connections to reach the cluster services
* Jobs creates one or more pods and as they complete successfully the job is marked as completed.
* Node is a worker machine in Kubernetes
* Namespaces are multiple virtual clusters backed by the same physical cluster
* Pods are the smallest deployable units of computing that can be created and managed in Kubernetes
* Persistent Volumes provides an API for users and administrators that abstracts details of how storage is provided from how it is consumed
* Replica Sets ensures that a specified number of pod replicas are running at any given time
* Secrets are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys
* Service Accounts provides an identity for processes that run in a Pod
* Services  is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service.
* Stateful Sets is the workload API object used to manage stateful applications.   
* and more...


![Relationship of pods, nodes, and containers](images/container-pod-node-master-relationship.jpg)

Kubernetes does not have the concept of an application. It has simple building blocks that you are required to compose. Kubernetes is a cloud native platform where the internal resource model is the same as the end user resource model.

# Recursos Chave

Um Pod é o menos modelo objeto que você pode criar e rodar. Você pode adicionar labels aos pods para identificar uma subseção para rodar operações. Quando você está pronto para escalar sua aplicação, você pode usar a label para dizer para o kubernetes qual pod você precisa escalar. Um pod representa um processo no seu cluster. Pods contem pelo menos um container que roda o trabalho e adicionalmente pode ter outros container chamados sidecars para monitoração, logs, etc. Essencialmente, um pod é um grupo de containers.

Quando nós falamos sobre uma aplicação, nós geralmente nos referiamos a um grupo de pods. Apesar de uma aplicação poder rodar em um único pod, nós geralmente construímos vários pods para conversarem uns com os outros para fazer uma aplicação útil. Nós vamos ver porque separar a aplicação logica e o backend database em pods separados que irão escalar melhor quando construímos uma aplicação breve.

Services definem como expor sua aplicação como uma entrada DNS para ter uma referência estável. Nós usamos seletor baseado em query para escolher qual pod fornecer esse serviço.

O usuário manipula diretamente os recursos via yaml:
`$ kubectl (create|get|apply|delete) -f myResource.yaml`

Kubernetes nos fornece com umas client interface através do ‘kubectl’. O comando kubectl te permite gerenciar suas aplicações, gerenciar seu cluster e os recursos do cluster, modificando o modelo dentro do data store.

# Kubernetes application deployment workflow

![deployment workflow](images/app_deploy_workflow.png)

1. User via "kubectl" deploys a new application. Kubectl sends the request to the API Server.
2. API server receives the request and stores it in the data store (etcd). Once the request is written to data store, the API server is done with the request.
3. Watchers detects the resource changes and send a notification to controller to act upon it
4. Controller detects the new app and creates new pods to match the desired number# of instances. Any changes to the stored model will be picked up to create or delete Pods.
5. Scheduler assigns new pods to a Node based on a criteria. Scheduler makes decisions to run Pods on specific Nodes in the cluster. Scheduler modifies the model with the node information.
6. Kubelet on a node detects a pod with an assignment to itself, and deploys the requested containers via the container runtime (e.g. Docker). Each Node watches the storage to see what pods it is assigned to run. It takes necessary actions on resource assigned to it like create/delete Pods.
7. Kubeproxy manages network traffic for the pods – including service discovery and load-balancing. Kubeproxy is responsible for communication between Pods that want to interact.


# Lab information

IBM Cloud provides the capability to run applications in containers on Kubernetes. The IBM Cloud Container Service runs Kubernetes clusters which deliver the following:

* Powerful tools
* Intuitive user experience
* Built-in security and isolation to enable rapid delivery of secure applications
* Cloud services including cognitive capabilities from Watson
* Capability to manage dedicated cluster resources for both stateless applications and stateful workloads


#  Lab overview

[Lab 0](Lab0) (Optional): Provides a walkthrough for installing IBM Cloud command-line tools and the Kubernetes CLI. You can skip this lab if you have the IBM Cloud CLI, the container-service plugin, the containers-registry plugin, and the kubectl CLI already installed on your machine.

[Lab 1](Lab1): This lab walks through creating and deploying a simple "guestbook" app written in Go as a net/http Server and accessing it.

[Lab 2](Lab2): Builds on lab 1 to expand to a more resilient setup which can survive having containers fail and recover. Lab 2 will also walk through basic services you need to get started with Kubernetes and the IBM Cloud Container Service

[Lab 3](Lab3): Builds on lab 2 by increasing the capabilities of the deployed Guestbook application. This lab covers basic distributed application design and how kubernetes helps you use standard design practices.

[Lab 4](Lab4): How to enable your application so Kubernetes can automatically monitor and recover your applications with no user intervention.

[Lab D](LabD): Debugging tips and tricks to help you along your Kubernetes journey. This lab is useful reference that does not follow in a specific sequence of the other labs. 
