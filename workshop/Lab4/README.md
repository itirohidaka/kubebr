# *** EM CONSTRUÇÃO ***

# 1. Verifique a saúde dos apps

O Kubernetes usa checagens de disponibilidade (liveness probes) para saber quando é o momento de reiniciar um container. Por exemplo, as liveness probes podem descobrir um deadlock; onde uma aplicação está rodando, porém está impossibilitada de fazer algum progresso. Reiniciar o container em tal estado pode ajudar a tornar a aplicação mais disponível apesar dos erros.

Além disso, o Kubernetes usa readiness checks para saber quando um container está pronto para começar a aceitar tráfego. Um pod é considerado pronto quando todos os seus containers estiverem prontos. Uma utilidade para essa verificação é controlar quais pods são usados como back-end para serviços. Quando um pod não está pronto, ele é removido dos balanceadores de carga.

Nesse exemplo, definimos uma liveness probe HTTP para checar a saúde do container a cada 5 segundos. Para os primeiros 10-15 seconds, o  `/healthz` retorna uma resposta `200` e falhará depois. O Kubernetes irá reiniciar o serviço automaticamente.

1. Abra o arquivo  `healthcheck.yml` com um editor de textos. Esse script de configuração combina alguns passos da lição anterior; para criar um deployment e um serviço ao mesmo tempo. Desenvolvedores de aplicação podem usar esses scripts quando são feitas atualizações, ou para diagnosticar problemas, recriando os pods:

   1. Atualize os detalhes para a imagem no seu private registry namespace:

      ```
      image: "ibmcom/guestbook:v2"
      ```

   2. Observe o liveness probe HTTP que verifica a saúde do container a cada 5 segundos.
      ```
      livenessProbe:
                  httpGet:
                    path: /healthz
                    port: 3000
                  initialDelaySeconds: 5
                  periodSeconds: 5
      ```

   3. Na seção **Service**, observe o  `NodePort`. Ao invés de gerar um nodeport aleatório, como você fez na lição anterior, você pode especificar uma porta no intervalo 30000 – 32767. Este exemplo usa 30072.

2.	Rode o script de configuração no cluster. Quando o deployment e o serviço estiverem criados, a aplicação estará disponível para que todos vejam:

   ```
   kubectl apply -f healthcheck.yml
   ```
   
   Agora que todo o trabalho no deployment está pronto, verifique como tudo acabou. Você pode notar que por haver mais instâncias rodando, as coisas podem ocorrer um pouco mais devagar.

3.	Abra um navegador e observe o app. Para formar a URL, combine o IP com o NodePort especificado no script de configuração. Para conseguir um IP público para o worker node:

   ```
   ibmcloud cs workers <cluster-name>
   ```


   Nos primeiros 10 - 15 segundos, uma mensagem “200” será retornada, e então saberá que a aplicação está rodando sem problemas. Após esses 15 segundos, uma mensagem de timeout será exibida.

4. Inicie o seu dashboard Kubernetes:

   1. Obtenha suas credenciais para o Kubernetes.
      
      ```
      kubectl config view -o jsonpath='{.users[0].user.auth-provider.config.id-token}'
      ```

   2. Copie o **id-token** exibido.    
   
   3. Defina o proxy com o número padrão de porta.

      ```
      kubectl proxy
      ```

      Saída:

      ```
      Starting to serve on 127.0.0.1:8001
      ```
   
   4. Entre no dashboard.
      
      1. Abra a seguinte URL em um navegador.
         
         ```
         http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
         ```
      
      2. Na página de entrada, selecione o método de autenticação **Token**.
 
      3. Então, cole o **id-token** que você copious anteriormente no campo **Token** e clique em **SIGN IN**.
  
   Na aba **Workloads** , você pode ver os recursos que criou. A partir desta aba, você pode atualizar contínuas vezes e ver que a verificação da saúde do ambiente está funcionando. Na seção **Pods** você pode ver quantas vezes os pods são reiniciados quando os containers dentro deles são recriados. É possível que erros apareçam no dashboard, indicando que a verificação encontrou algum problema. Espere alguns minutos antes de dar um novo refresh na página. Você verá o número de reinicializações e mudanças em cada pod.

5. 	Preparado para apagar o que você criou antes de continuar? Desta vez, você pode usar o mesmo script de configuração para excluir ambos os recursos que você criou.

   ```kubectl delete -f healthcheck.yml```

6.	Quando você terminar de explorer o dashboard do, na sua CLI, pression `CTRL+C` para sair do comando  `proxy`.
