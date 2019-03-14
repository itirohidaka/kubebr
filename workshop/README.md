# IBM Cloud Kubernetes Service Lab

<img src="https://kubernetes.io/images/favicon.png" width="200">

# Uma introdução à containers

Ei, você está procurando um curso 101 sobre containers? Confira nosso [Docker Essentials](https://developer.ibm.com/courses/all/docker-essentials-extend-your-apps-with-containers/).

Containers permitem que você rode aplicações isoladas com segurança e cotas nos recursos do sistema. Containers começaram como uma ferramenta individual entregue com kernel linux. Docker foi lançado tornando containers fácil de se usar e os desenvolvedores rapidamente aderiram essa ideia. Containers também despertaram interesses em arquiteturas de microsserviços, uma metodologia para desenvolvimento de aplicações aonde aplicações complexas são divididas em partes e essas partes trabalham juntas.

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

O isolamento no linux é fornecido pela feature chamada 'namespaces'. Cada diferente tipo de isolamento (usuário IE, cgroups) é fornecido por um diferente namespace.

Essa é uma lista de alguns namespaces que são normalmente usados e visíveis ao usuário:

* PID - IDs dos processos
* USER - IDs de grupos e usuários
* UTS - hostname e nome de domínio
* NS - mount points
* NET - dispositivos de rede, stacks e portas
* CGROUPS - limites de controle e monitoramento de recursos

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

# Kubernetes e containers: Overview

Vamos discutir sobre orquestração Kubernetes para containers antes de construirmos aplicações nele. Nós precisamos entender os seguintes fatos sobre isso:

* O que extamente é o Kubernetes?
* Como o Kubernetes foi cirado?
* Arquitetura do Kubernetes
* Modelo de recurso do Kubenetes
* Kubernetes na IBM
* Let's get started

# O que é Kubernetes?

Agora que nós sabemos o que são containers, vamos definir o que é o Kubernetes. Kubernetes é um orquestrador de containers para provisionar, gerenciar e escalar aplicações. Em outras palavras, Kubernetes permite que você gerencie o ciclo de vida de aplicações conteinerizadas dentro de um cluster de nodes (que é uma coleção de máquina workers, por exemplo, VMs, máquinas físicas etc.).


Sua aplicação pode precisar de outros recursos como volumes, redes e secrets que vão te ajudar a fazer coisas como conexão com banco de dados, conversar com backend com firewall e chaves de segurança. Kubernetes ajuda você a adicionar esses recursos na sua aplicação. Recursos de infraestrutura necessários da aplicação são gerenciados delcarativamente.

**Observação:** Mesos e Swarm são outras tecnologias de orquestração.  

O paradigma chave é o modo declarativo. O usuário fornece o estado desejado enquanto o Kubenetes fará tudo possível para atender. Se você precisar de 5 instâncias, você não inicia 5 instâncias separadas por conta própria mas diz para o kubernetes que você precisa de 5 instâncias e o kubernetes vai atender esse estado automaticamente. Simplesmente nesse momento você precisa saber que você declara o estado que você quer e o Kubernetes fará isso acontecer. Se alguma coisa der errado com uma das instâncias e a instância cair, Kubernetes ainda saberá o estado desejado e criará novas instâncias no node disponível.

**Fun fact** Kubernetes pode ser chamado por muitos nomes. As vezes é encurtado como _k8s_ (perdendo 8 letras), ou carinhosamente chamado de _kube_. A palavra vem na Grécia antiga que significa "Helmsman" (timoneiro). O helmsman é a pessoa responsável por dirigir o barco. Esperamos que você tenha percebido a analogia entra dirigir um barco e as decisões feitas para orquestrar um container em um cluster.

# Como o Kubernetes foi criado?

Google queria abrir ao público seu conhecimento de criação e execução da ferramenta interna Borg & Omega. Ela adotou Open Governance para Kubernetes fundando a Cloud Native Computing Foundation (CNCF) e doando o Kubernetes para essa fundação, assim tornando Kubernetes menos influente pelo Google. Muitas companias como a RedHat, Microsoft, IBM e Amazon se juntaram rapidamente a fundação.
Ponto de entrada principal para o projeto Kubernetes está em [http://kubernetes.io](http://kubernetes.io) e o código fonte pode ser encontrado em [https://github.com/kubernetes](https://github.com/kubernetes).

# Arquitetura do Kubenetes

Em sua essência, Kubernetes é um data store (etcd). O modo declarativo é armazenado dentro do data store como objeto, que significa que quando você diz que quer 5 instâncias de um container, aquele request é armazenado no data store. Essa mudança de informação é vigiada e delegada para os controllers que tomam a ação. Os Controllers reagem então ao modelo e tentam tomar uma ação para atingir ao estado desejado. O poder do Kubernetes está no modelo simplista.

Como monstrado, API server é um simples manipulador de HTTP server que realiza operações como criação/leitura/atualização/exlusão(CRUD) no data store. Então o controller pega a mudança que você deseja e realiza aquela mudança. Controllers são responsáveis por instanciar os recursos atuais representados por qualquer recurso do Kubernetes. Esses recursos atuais são o que a sua aplicação precisa para permitir que execute perfeitamente.

![architecture diagram](images/kubernetes_arch.png)

# Modelo de recursos do Kubernetes

A infraestrutura do kubernetes define um recurso para cada  defines a resource for every propósito. Cada recurso é monitorado e processado pelo controller. Quando você define sua aplicação, ela contem uma coleção desses recursos. Essa coleção será então lida pelos Controllers para criar as instâncias de suporte do seu aplicativo. Alguns recursos que você pode trabalhar estão listados abaixo para sua referência, para uma lista completa acesse [https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/). Nessa aula nós vamos usar alguns deles como Pod, Deployment, etc.

* Config Maps holds configuration data for pods to consume.
* Conjuntos de Daemon asseguram que cada node no cluster execute este Pod
* Deployments definem um estado desejado de um objeto deployment
* Events fornecem um ciclo de vida de events em Pods e outros objetos deployment
* Endpoints permitem conexão de entrada para alcançar os services do cluster
* Ingress é um conjunto de regras que permite conexões de entraga para alcançar services de um cluster
* Jobs cria um ou mais pods a medida que completam com sucesso o job, é marcado como completo.
* Node é uma máquina trabalhadora em Kubernetes
* Namespaces são vários clusters virtuais apoiados em um mesmo cluster físico
* Pods são a menor unidade de computing que pode ser criada e gerenciada no Kubernetes
* Persistent Volumes fornecem uma API para usuários e administradores que abstrai detalhes de como o storage é forncecido e como é consumido
* Replica Sets garantem que um número específico de replicas de pods estejam rodando a qualquer momento
* Secrets são projetados para armazenar informações sensíveis, assim como senhas, OAuth tokens e chaves ssh
* Service Accounts fornecem uma identidade para processos que rodam em um pod
* Services é uma abstração que definem um conjunto lógico de Pods e uma policy pela qual acessá-los - as vezes chamados de micro-service.
* Stateful Sets é o objeto API do workload usado para gerenciar aplicações stateful.   
* e mais...


![Relationship of pods, nodes, and containers](images/container-pod-node-master-relationship.jpg)

Kubernetes não tem o conceito de uma aplicação. Tem blocos de construção simples que você precisa para criar. Kubernetes é uma plataforma nativa de cloud onde o modelo de recurso interno é o mesmo que o modelo de recursos do usuário final..

# Recursos Chave

Um Pod é o menos modelo objeto que você pode criar e rodar. Você pode adicionar labels aos pods para identificar uma subseção para rodar operações. Quando você está pronto para escalar sua aplicação, você pode usar a label para dizer para o kubernetes qual pod você precisa escalar. Um pod representa um processo no seu cluster. Pods contem pelo menos um container que roda o trabalho e adicionalmente pode ter outros container chamados sidecars para monitoração, logs, etc. Essencialmente, um pod é um grupo de containers.

Quando nós falamos sobre uma aplicação, nós geralmente nos referiamos a um grupo de pods. Apesar de uma aplicação poder rodar em um único pod, nós geralmente construímos vários pods para conversarem uns com os outros para fazer uma aplicação útil. Nós vamos ver porque separar a aplicação logica e o backend database em pods separados que irão escalar melhor quando construímos uma aplicação breve.

Services definem como expor sua aplicação como uma entrada DNS para ter uma referência estável. Nós usamos seletor baseado em query para escolher qual pod fornecer esse serviço.

O usuário manipula diretamente os recursos via yaml:
`$ kubectl (create|get|apply|delete) -f myResource.yaml`

Kubernetes nos fornece com umas client interface através do ‘kubectl’. O comando kubectl te permite gerenciar suas aplicações, gerenciar seu cluster e os recursos do cluster, modificando o modelo dentro do data store.

# Workflow do deployment de uma aplicação no Kubernetes

![deployment workflow](images/app_deploy_workflow.png)

1. O usuário faz o deploy de uma nova aplicação através do "kubectl". Kubectl envia o request para o API server.
2. O API server recebe o request e armazena no data store (etcd). Uma vez com o request armazenado no data store,  o API server encerra o request.
3. Os Watchers detecta as mudanças de recursos e envia uma notificação ao controller para agir sobre isso
4. O Controller detecta o novo app e cria novos pods para corresponder o número desejado de instâncias. Qualquer mudança no modelo armazenado, serão selecionadas para criar ou deletar Pods.
5. O Scheduler atribui novos pods para um Node baseado nos critérios. Scheduler faz as decisões para rodar Pods em Nodes específicos em um cluster. O Scheduler modifica o modelo com a informação do node.
6. Kubelet em um node detecta um pod com atribuído para ele mesmo, e faz o deploy dos containers requisitados via container runtime (e.g. Docker). Cada node vigia o storage para ver quais pods foram atribuídos para ele rodar. Ele executa ações necessárias no recurso atribuído a ele como criar/deletar pods.
7. Kubeproxy gerencia o tráfego de rede para os pods – incluindo service discovery e load-balancing. Kubeproxy é responsável pela comunicação entre Pods que querem interagir.

# Informações do Lab

IBM Cloud fornece a capacidade de rodar aplicações em containers no Kubernetes. O IBM Cloud Kubernetes Service roda um cluster Kubernetes que te entrega o seguinte:

* Ferramentas poderosas
* Experiência intuitiva do usuário
* Segurança e isolamento construídos para permitir entrega rápida de aplicações seguras
* Serviços da Cloud incluindo capacidades cognitivas do Watson
* Capacidade de gerenciar recursos de um cluster dedicado para aplicações stateless e stateful workloads


#  Overview do Lab

[Lab 0](Lab0) (Optional): Fornece uma apresentação de como instalar as ferramentas da IBM Cloud command-line e a CLI do Kubernetes. Você pode pular esse lab se você já tiver instalado a CLI IBM CLoud, plugin do kubernetes service, plugin do container registry e a CLI do Kubernetes.

[Lab 1](Lab1): Esse lab percorre a criação e o deploy de um simples app "guestbook" escrito em Go como um net/http server e como acessá-lo.

[Lab 2](Lab2): Feito a partir do lab 1 para expandir para um setup mais resiliente que pode sobreviver à containers com falha e recover. Lab 2 também percorre serviços básicos que você precisa para começar com Kubernetes e o IBM Cloud Kubernetes Service (IKS)

[Lab 3](Lab3): Feito a partir do lab 2 para aumentar as capacidades da aplicação Guestbook, já com o deploy feito. Esse lab cobre o design básico de aplicativos distrubuídos e como o Kubernetes te ajuda a usar práticas padrões de design.

[Lab 4](Lab4): Como permitir que sua aplicação e o Kubernetes possam automaticamente monitorar e recuperar a aplicação sem intervenção do usuário.

[Lab DevOps](LabDevOps): Nesse lab você verá como é fácil fazer o deploy de uma aplicação em Kubernetes utilizando a delivery pipeline da IBM.

[Lab D](LabD): Dicas de Debugging e truques para te ajudar em sua jornada de Kubernetes. Esse lab é uma referência útil que não segue uma sequência específica dos outros Labs. 
