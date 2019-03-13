# Lab 5: Fazendo deploy de uma aplicação utilizando IBM Toolchain


Nesse lab, você aprenderá como fazer o deploy de uma aplicação utilizando a IBM Toolchain. 

Clone o repositório:

```
$ git clone https://github.com/IBM/guestbook.git
```


# 1. Criando sua Toolchain

Uma toolchain é um grupo de ferramentas integradas para desenvolvimento, monitoração e mais. Você pode criar sua toolchain do zero ou se basear em um template.

Faça login em [cloud.ibm.com](https://cloud.ibm.com), acesse a sessão de DevOps do portal. Iremos criar nossa toolchain baseada no template para deploy em Kubernetes. Preencha os campos e crie sua toolchain.

Esse template vem com 3 tools:
`Git Repos`
Para pegar a aplicações do repositório Git

`Eclipse Orion Web IDE`
Com esssa tool você é capaz editar sua aplicação a partir do portal

`Delivery Pipeline`
Esteira de deploy com 3 stages, Build, Validate e Deploy.

# 2. Configurando sua Toolchain

Esse template utiliza o repositório GitLab da IBM, precisaremos adicionar uma tool de github para fazermos o deploy do guestbook. Delete a tool code do GitLab e selecione `Add a Tool` para adicionar a tool do GitHub, agora precisaremos configurá-la. 
