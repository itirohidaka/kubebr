# Lab 0: IBM Cloud Kubernetes Service


Antes de começar o seu aprendizado, você precisará instalar as CLIs necessárias para criar e gerenciar seu cluster Kubernetes no IBM Cloud Kubernetes Service, e também para implementar apps encapsulados no seu cluster.

Neste lab, você encontrará informações para a instalação das seguintes CLIs e plug-ins:

* IBM Cloud CLI, Versão 0.5.0 ou posterior
* IBM Cloud Kubernetes Service plug-in
* Kubernetes CLI, Versão 1.10.8 ou posterior

Caso tenha as CLIs e plug-ins já instalados, você pode pular esta etapa e seguir para a próxima.

# Instalando a IBM Cloud command-line interface

1. Como pré requisito para o plug-in IBM Cloud Container Service, instale a [IBM Cloud command-line interface](https://clis.ng.bluemix.net/ui/home.html). Uma vez instalada, você pode acessar a IBM Cloud a partir da sua command-line utilizando o prefixo `ibmcloud`.
2. Para realizar o login na IBM Cloud CLI: `ibmcloud login`.
3. Insira sua credencial da IBM Cloud.

   **Note:** Se você possui um ID federado, use `ibmcloud login --sso` para realizar o login na IBM Cloud CLI. Insira seu nome, e utilize a URL disponibilizada na sua CLI para adquirir uma senha de utilização única. Você sabe que utiliza um ID federado no momento em que o login falha sem o parâmetro `--sso` e funciona normalmente com o `--sso`.

# Instalando o IBM Cloud Kubernetes Service plug-in
1. Para criar um cluster Kubernetes e gerenciar worker nodes, instale o IBM Cloud Container Service plug-in:
   ```ibmcloud plugin install container-service -r Bluemix```

   **Note:** O prefixo para rodar commandos utilizando o IBM Cloud Container Service plug-in é `ibmcloud cs`.

2. Para verificar se a instalação do plug-in foi feita corretamente, rode o seguinte comando:
```ibmcloud plugin list```

   IBM Cloud Container Service plug-in será exibido nos resultados como `container-service`.

# Download da Kubernetes CLI

Para ter uma versão local do Kubernetes dashboard e para implementar apps em seu cluster, você precisará instalar a Kubernetes CLI correspondente ao seu sistema operacional:

* [OS X](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/darwin/amd64/kubectl)
* [Linux](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/linux/amd64/kubectl)
* [Windows](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/windows/amd64/kubectl.exe)

**Para usuários Windows:** Instale a Kubernetes CLI no mesmo diretório que a IBM Cloud CLI. Essa configuração facilita algumas mudanças em caminhos de arquivo quando você roda alguns comandos posteriormente.

**Para usuários OS X e Linux:**

1. Mova o arquivo executável para o diretório `/usr/local/bin` usando o comando `mv /<path_to_file>/kubectl /usr/local/bin/kubectl` .

2. Certifique-se que o caminho `/usr/local/bin` esteja listado em sua variável de sistema PATH.
```
$echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

3. Converta o arquivo binário para um arquivo executável:  `chmod +x /usr/local/bin/kubectl`

# Download do Código-fonte
O repositório `guestbook` contém a aplicação que iremos implementar, e iremos utilizar os arquivos de configuração deste repositório. A aplicação Guestbook tem duas versões; v1 e v2, que iremos utilizar para demonstrar algumas funcionalidades posteriormente. Todos os aquivos de configuração que usamos estão no diretório guestbook/v1.

O repositório `kube101` contém as instruções passo a passo para rodar o workshop.
```console
$ git clone https://github.com/IBM/guestbook.git
$ git clone https://github.com/itirohidaka/kube101.git
```
