# Lab 1. Prepare e faça o deploy da sua primeira aplicação

Aprenda como fazer o deploy de uma aplicação em um cluster Kubernetes dentro do IBM Kubernetes Service

# 0. Instale a CLI e provisione um cluster Kubernetes

Se você ainda não possuir:
1. Instale as CLIs da IBM Cloud e faça o login, como descrito no  [Lab 0](../Lab0/README.md).
2. Provisione um cluster:

   ```$ ibmcloud cs cluster-create --name <name-of-cluster>```

Uma vez o cluster provisionado, a CLI do kubernetes `kubectl` precisa ser configurada para conversar com o cluster provisionado.

1. Execute `$ ibmcloud cs cluster-config <name-of-cluster>`, e configure a variável de ambiente `KUBECONFIG`
   baseado na saída do comando. Isso fará seu client `kubectl` apontar para seu cluster Kubernetes.

Uma vez com seu client configurado, você está pronto para fazer o deploy da sua primeira aplicação, `guestbook`.

# 1. Deploy da sua aplicação

Nessa parte do lab nós faremos o deploy de uma aplicação chamada `guestbook`,
que já foi construída e disponibilizada no DockerHub como 
`ibmcom/guestbook:v1`.

1. Execute o `guestbook` rodando o seguinte comando:

   ```$ kubectl run guestbook --image=ibmcom/guestbook:v1```

   Esse comando pode levar um tempo. Para checar o status da aplicação em execução, 
você pode rodar  `$ kubectl get pods`.

   Você irá se deparar com uma saída parecida com essa:

   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    0/1       ContainerCreating   0          1m
   ```
   Após um tempo o status deverá estar como `Running`.
   
   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
   ```
   
   O resultado final não é somente um pod contendo nossos containers da aplicação, 
mas também o recurso chamado Deployment que gerencia o ciclo de vida desses pods.
 
   
3. Assim que os status estiverem como `Running`, nós precisamos expor esse Deployment
   como um serviço para que consigamos acessá-lo através do IP do worker node.
   A aplicação `guestbook` atende na porta 3000.  Execute:

   ```console
   $ kubectl expose deployment guestbook --type="NodePort" --port=3000
   service "guestbook" exposed
   ```

4. Para encontrar a porta usada nesse worker node, examine seu novo serviço: 

   ```console
   $ kubectl get service guestbook
   NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   guestbook   NodePort   10.10.10.253   <none>        3000:31208/TCP   1m
   ```
   
   Podemos ver que nossa `<nodeport>` é a `31208`. Nós podemos verificar também o mapeamento da porta 3000
   dentro do pod exposto para o cluster na porta 31208. Essa porta no range 31000 é escolhida automaticamente, 
   e pode ser diferente da sua.

5. Agora, a aplicação `guestbook` está rodando no seu cluster e exposta para internet. Precisamos descobrir como acessá-la.
   O worker node que está rodando no kubernetes service pega um endereço IP externo.
   Execute o comando `$ ibmcloud cs workers <name-of-cluster>`, e note que o IP público é listado na linha `<public-IP>`.
   
   ```console
   $ ibmcloud cs workers osscluster
   OK
   ID                                                 Public IP        Private IP     Machine Type   State    Status   Zone    Version  
   kube-hou02-pa1e3ee39f549640aebea69a444f51fe55-w1   173.193.99.136   10.76.194.30   free           normal   Ready    hou02   1.5.6_1500*
   ```
   
   Podemos ver que nosso `<public-IP>` é `173.193.99.136`.
   
6. Agora que você possui o IP e a porta, você pode acessar a aplicação no seu browser com seguinte endereço
  `<public-IP>:<nodeport>`. Nesse exemplo seria `173.193.99.136:31208`.
   
Parabéns, você fez o deploy de uma aplicação no Kubernetes!

Quando você tiver terminado, você pode também usar esse deployment
[próximo lab deste curso](../Lab2/README.md), ou você pode remover o deployment e encerrar o curso por aqui

  1. Para remover o deployment, execute `$ kubectl delete deployment guestbook`.

  2. Para remover o serviço, execute  `$ kubectl delete service guestbook`.

Agora você voltar na raiz do repositório para se preparar para o próximo Lab: `$ cd ..`.
