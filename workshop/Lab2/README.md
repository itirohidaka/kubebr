# Lab 2: Escale e atualize Deployments

Neste lab, você irá aprender como atualizar o número de instâncias de um deployment e como disponibilizar uma atualização da sua aplicação em Kubernetes de forma segura.

Para este lab, você precisará de um deployment ativo da aplicação `guestbook`, do lab anterior. Caso você tenha apagado, recrie utilizando:

```console
$ kubectl run guestbook --image=ibmcom/guestbook:v1
```
    
# 1.Escale aplicações utilizando réplicas

Uma *replica* é uma cópia de um pod que contém um serviço rodando. Tendo múltiplas replicas de um pod, você garante que seu deployment terá os recursos disponíveis para aguentar uma carga crescente na sua aplicação.

1. `kubectl` providencia um subcomando `scale` ara mudar o tamanho de um deployment existente. Vamos aumentar nossa capacidade; de uma instância de`guestbook` para 10 instâncias:

   ``` console
   $ kubectl scale --replicas=10 deployment guestbook
   deployment "guestbook" scaled
   ```

   O Kubernetes agora irá tentar chegar ao estado desejado de 10 replicas, subindo 9 novos pods com a mesma configuração.

2. Para ver as mudanças acontecendo, você pode rodar:
   `kubectl rollout status deployment guestbook`.

   O rollout pode acontecer de forma tão rápida, que talvez as seguintes mensagens _não_ sejam exibidas:

   ```console
   $ kubectl rollout status deployment guestbook
   Waiting for rollout to finish: 1 of 10 updated replicas are available...
   Waiting for rollout to finish: 2 of 10 updated replicas are available...
   Waiting for rollout to finish: 3 of 10 updated replicas are available...
   Waiting for rollout to finish: 4 of 10 updated replicas are available...
   Waiting for rollout to finish: 5 of 10 updated replicas are available...
   Waiting for rollout to finish: 6 of 10 updated replicas are available...
   Waiting for rollout to finish: 7 of 10 updated replicas are available...
   Waiting for rollout to finish: 8 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   deployment "guestbook" successfully rolled out
   ```

3. Uma vez finalizado o rollout, confira se os seus pods estão ativos usando o seguinte comando: 
   `kubectl get pods`.

   Você verá uma listagem com 10 réplicas do seu deployment:

   ```console
   $ kubectl get pods
   NAME                        READY     STATUS    RESTARTS   AGE
   guestbook-562211614-1tqm7   1/1       Running   0          1d
   guestbook-562211614-1zqn4   1/1       Running   0          2m
   guestbook-562211614-5htdz   1/1       Running   0          2m
   guestbook-562211614-6h04h   1/1       Running   0          2m
   guestbook-562211614-ds9hb   1/1       Running   0          2m
   guestbook-562211614-nb5qp   1/1       Running   0          2m
   guestbook-562211614-vtfp2   1/1       Running   0          2m
   guestbook-562211614-vz5qw   1/1       Running   0          2m
   guestbook-562211614-zksw3   1/1       Running   0          2m
   guestbook-562211614-zsp0j   1/1       Running   0          2m
   ```

**Tip:** Outra forma de melhorar a disponibilidade é
[adicionar clusters e regiões](https://console.bluemix.net/docs/containers/cs_planning.html#cs_planning_cluster_config)
ao seu deployment, como mostrado no seguinte diagrama:

![HA with more clusters and regions](../images/cluster_ha_roadmap.png)

# 2. Roll back e atualização de apps

O Kubernetes te permite atualizar uma aplicação sem a necessidade de que ela seja interrompida; isso facilita a mudança de um aplicativo já em execução e também permite desfazer uma atualização já feita, caso seja descoberto algum problema durante ou após a implementação.

No lab anterior, nós utilizamos uma imagem com a tag `v1`. Para a nossa atualização, nós vamos usar a imagem com a tag `v2`.

Para realizar o update e o roll back:   
1. Utilizando `kubectl`, você pode atualizar o seu deployment para que ele use a imagem
   `v2`. `kubectl` permite que você altere detalhes sobre recursos existentes, com o subcomando `set`. Podemos usá-lo para trocar a imagem que está sendo utilizada.
   
    ```$ kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2```

   Note que um Pod pode ter múltiplos containers, cada um com o seu próprio nome. Cada imagem pode ser trocada individualmente; ou todas de uma só vez, referenciando seu nome. No caso do nosso deployment `guestbook` Deployment, o nome do container também é `guestbook`.
   Múltiplos containers podem ser atualizados de uma só vez.
   ([Mais Informações](https://kubernetes.io/docs/user-guide/kubectl/kubectl_set_image/).)

3. Rode o comando  `kubectl rollout status deployment/guestbook` para checar o status do rollout. O rollout pode acontecer tão        rapidamente, que talvez as seguintes mensagens _não_ apareçam:
  
  
   ```console
   $ kubectl rollout status deployment/guestbook
   Waiting for rollout to finish: 2 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 3 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 4 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 5 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 6 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 7 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 8 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 9 out of 10 new replicas have been updated...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 1 old replicas are pending termination...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   deployment "guestbook" successfully rolled out
   ```

4. Teste a aplicação, como anteriormente, acessando `<public-IP>:<nodeport>` 
   no navegador, para confirmar que seu novo código está ativo.
   Lembre-se, para conseguir o "nodeport" e o "public-ip" utilize:

   `$ kubectl describe service guestbook`
   e
   `$ ibmcloud cs workers <name-of-cluster>`

   Para certificar-se de que você está rodando a "v2" do guestbook, olhe no título da página; deverá ser exibido como `Guestbook - v2`

5. Caso você queira desfazer o seu último rollout, utilize:
   ```console
   $ kubectl rollout undo deployment guestbook
   deployment "guestbook"
   ```

   Você poderá então usar `kubectl rollout status deployment/guestbook` para ver o status.
   
6. Quando um rollout é feito, você vê referências à *old* replicas e *new* replicas.
   Como as *old* replicas são os 10 pods originais, implementados quando nós escalamos a aplicação. *new* replicas vêm dos pods criados recentemente, com a imagem diferente. Todos estes pods são pertencentes ao deployment. O deployment gerencia esses dois sets de pods, utilizando um recurso chamado ReplicaSet. Podemos observar os ReplicaSets do guestbook com:
   ```console
   $ kubectl get replicasets -l run=guestbook
   NAME                   DESIRED   CURRENT   READY     AGE
   guestbook-5f5548d4f    10        10        10        21m
   guestbook-768cc55c78   0         0         0         3h
   ```

Antes de continuarmos, vamos apagar a aplicação, e então nós Podemos aprender diferentes formas de chegar ao mesmo resultado:

Para remover o deployment, utilize `kubectl delete deployment guestbook`.

Para remover o serviço, utilize `kubectl delete service guestbook`.
Parabéns! Você implementou a segunda versão da aplicação. O lab 2 está completo
