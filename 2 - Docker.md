-------------------------------------------------------------------
# O que é Docker e seu sistema
-------------------------------------------------------------------
## As tecnologias do Docker
O Docker nada mais é do que uma coleção de tecnologias para facilitar o deploy e a execução das nossas aplicações. A sua principal tecnologia é a Docker Engine, a plataforma que segura os containers, fazendo o intermédio entre o sistema operacional.

Outras tecnologias do Docker que facilitam a nossa vida e que veremos neste curso são o Docker Compose, um jeito fácil de definir e orquestrar múltiplos containers; o Docker Swarm, uma ferramenta para colocar múltiplos docker engines para trabalharem juntos em um cluster; o Docker Hub, um repositório com mais de 250 mil imagens diferentes para os nossos containers; e a Docker Machine, uma ferramenta que nos permite gerenciar o Docker em um host virtual.

## Open Source
Quando a empresa dotCloud tornou-se a Docker, Inc., focada em manter o Docker, ela o abriu para o mundo open source, tudo disponibilizado no seu GitHub, inclusive com várias empresas contribuindo para o desenvolvimento dessa tecnologia.

Apesar de haver alguns serviços pagos, em sua grande parte a tecnologia do Docker é uma tecnologia open source, utilizada por várias empresas. Então, vamos colocar as mãos na massa e aprender a instalar o Docker nas próximas aulas.
-------------------------------------------------------------------
# A diferença entre imagens e containers
-------------------------------------------------------------------
Primeiramente, devemos verificar se o Docker está rodando. Se sim, no Linux ou no macOs, vamos executar os comandos do Docker através do próprio terminal nativo do sistema operacional. No caso do Windows, o próprio Docker recomenda que os seus comandos sejam executados através do Windows PowerShell.
> OBS: Lembrando que se o seu Docker foi instalado pelo Docker Toolbox, você deve executar os seus comandos através do Docker Quickstart Terminal, terminal que foi instalado pelo próprio Docker Toolbox.

Para executar a imagem hello-world, executamos o comando docker run hello-world. Quando executado esse comando, a primeira coisa que o Docker faz é verificar se temos a imagem hello-world no nosso computador, e caso não tenhamos, o Docker buscará e baixará essa imagem no Docker Hub/Docker Store.

A imagem é como se fosse uma receita de bolo, uma série de instruções que o Docker seguirá para criar um container, que irá conter as instruções da imagem, do hello-world. Criado o container, o Docker executa-o. Então, tudo isso é feito quando executamos o docker run hello-world.

Há milhares de imagens na Docker Store disponíveis para executarmos a nossa aplicação. Por exemplo, temos a imagem do Ubuntu:

	docker run ubuntu

Ao executar o comando, o download começará. Podemos ver que não é feito somente um download, pois a imagem é dividida em camadas, que veremos mais à frente. Terminados os downloads, nenhuma mensagem é exibida, então significa que o container não foi criado? Na verdade o container foi criado, o que acontece que é a imagem do Ubuntu não executa nada, por isso nenhuma mensagem foi exibida.

Podemos verificar isso vendo os containers que estão sendo executados no momento, executando o seguinte comando:

	docker ps
	
Ao executar esse comando, vemos que não há nenhum container ativo, pois quando não há nada para o container executar, eles ficam parados. Para ver todos containers, inclusive os parados, adicionamos a flag -a ao comando acima:

	docker ps -a
```	
CONTAINER ID    IMAGE         COMMAND       CREATED         STATUS                     PORTS     NAMES
4139842e283a    ubuntu        "/bin/bash"   3 minutes ago   Exited (0) 3 minutes ago             elastic_albattani
c1a155091114    hello-world   "/hello"      4 days ago      Exited (0) 4 days ago                nifty_mcclintock
```

Com esse comando, conseguimos ver o id e o nome do container, valores que são criados pelo próprio Docker. Além disso, temos a outras informações dos containers, a imagem em que eles são baseados, o comando inicial que roda quando ele é executado, quando ele foi criado e qual o seu status.

Então, o container do Ubuntu foi executado, mas ele não fez nada pois não pedimos para o container executar algo que funcione dentro do Ubuntu. Então, quando executamos o container do Ubuntu, precisamos passar para ele um comando que rode dentro dele, por exemplo:

	docker run ubuntu echo "Ola Mundo"
	
Com isso, o Docker irá executar um container com Ubuntu, executar o comando echo "Ola Mundo" dentro dele e nos retornar a saída:

	alura@alura-estudio-03:~$ docker run ubuntu echo "Ola Mundo"
Ola Mundo

Só que sabemos que o Ubuntu é um sistema operacional completo, então não queremos ficar somente executando um comando por comando dentro dele, sempre criando um novo container. Então, como fazemos para criar um container e interagir com ele mais do que com um único comando?

## Trabalhando dentro de um container

Podemos fazer com que o terminal da nossa máquina seja integrado ao terminal de dentro do container, para ficar um terminal interativo. Podemos fazer isso adicionando a flag -it ao comando, atrelando assim o terminal que estamos utilizando ao terminal do container:

	docker run -it ubuntu
	
Assim que executamos o comando, já podemos perceber que o terminal muda:

	alura@alura-estudio-03:~$ docker run -it ubuntu root@05025384675e:/# 

Com isso, estamos trabalhando dentro do container. E dentro dele, podemos trabalhar como se estivéssemos trabalhando dentro do terminal de um Ubuntu, executando comandos como ls, cat, etc.

Agora, se abrirmos outro terminal e executar o comando docker ps, veremos o container que estamos executando. Podemos parar a sua execução, digitando no container o comando exit ou através do atalho CTRL + D .

## Executando novamente um container

Paramos a execução do container, tanto que o comando docker ps não nos retorna mais nada. E se listarmos todos os containers, através do comando docker ps -a, vemos que ele está lá, parado. Mas agora, para não criar novamente um novo container, queremos executá-lo novamente.

Fazemos isso pegando id do container a ser iniciado, e passando-o ao comando docker start

	docker start Id_do_container
	
Esse comando roda um container já criado, mas não atrela o nosso terminal ao terminal dele. Para atrelar os terminais, primeiramente devemos parar o container, com o comando docker stop mais o seu id:

	docker stop 05025384675e
	
E rodamos novamente o container, mas passando duas flags: -a, de attach, para integrar os terminais, e -i, de interactive, para interagirmos com o terminal, para podermos escrever nele:

	alura@alura-estudio-03:~$ docker start -a -i 05025384675e root@05025384675e:/# 

Com isso, conseguimos ver um pouco de como subir um container, pará-lo e executá-lo novamente, além de trabalhar dentro dele.

Os dois principais estados de um container, quando criamos um ou iniciamos, ele fica no estado de running, e quanto a sua execução encerra ou paramos, ele fica no estado de stopped:

<img src="https://s3.amazonaws.com/caelum-online-public/646-docker/02/imagens/estados-container.png" />

## Removendo containers

Só que com os testes que fizemos até agora, acabamos criando vários containers (lembrando que podemos ver todos os containers criados executando o comando docker ps -a) e nunca removemos algum deles, já que os comandos acima só mudam os seus estados. Para remover um container, executamos o comando docker rm, passando para ele o id do container a ser removido, por exemplo:

	docker rm c9f83bfb82a8

Mas para limpar todos os containers inativos, devemos remover um por um? Não, pois há um novo comando do Docker, o prune, que serve para limparmos algo específico do Docker. Como queremos remover os containers parados, executamos o seguinte comando:

	docker container prune
	
O comando é tão poderoso que ele pede para confirmarmos se é isso mesmo que queremos fazer.

## Listando e removendo imagens

E do mesmo jeito que temos o comando docker container para mexermos com o container, temos o comando docker images, que nos exibe as imagens que temos na nossa máquina. Para remover uma imagem, utilizarmos o comando docker rmi, passando para ele o nome da imagem a ser removida, por exemplo:

	docker rmi hello-world
	
## Camadas de uma imagem

Quando baixamos a imagem do Ubuntu, reparamos que ela possui camadas, mas como elas funcionam? Toda imagem que baixamos é composta de uma ou mais camadas, e esse sistema tem o nome de Layered File System.

Essas camadas podem ser reaproveitadas em outras imagens. Por exemplo, já temos a imagem do Ubuntu, isso inclui as suas camadas, e agora queremos baixar a imagem do CentOS. Se o CentOS compartilha alguma camada que já tem na imagem do Ubuntu, o Docker é inteligente e só baixará as camadas diferentes, e não baixará novamente as camadas que já temos no nosso computador:


<img src="https://s3.amazonaws.com/caelum-online-public/646-docker/02/imagens/camadas.png" />

No caso da imagem acima, o Docker só baixará as duas primeiras camadas da imagem do CentOS, já que as duas últimas são as mesmas da imagem do Ubuntu, que já temos na nossa máquina. Assim poupamos tempo, já que precisamos de menos tempo para baixar uma imagem.

Uma outra vantagem é que as camadas de uma imagem são somente para leitura. Mas como então conseguimos criar arquivos na aula anterior? O que acontece é que não escrevemos na imagem, já que quando criamos um container, ele cria uma nova camada acima da imagem, e nessa camada podemos ler e escrever:

<img src="https://s3.amazonaws.com/caelum-online-public/646-docker/02/imagens/container-layer.png"/>

Então, quando criamos um container, ele é criado em cima de uma imagem já existente e nele nós conseguimos escrever. E com uma imagem base, podemos reaproveitá-la para diversos containers:

<img src="https://s3.amazonaws.com/caelum-online-public/646-docker/02/imagens/imagem-varios-containers.png" />

Isso nos traz economia de espaço, já que não precisamos ter uma imagem por container.

Imagens do Docker possuem um sistema de arquivos em camadas (Layered File System) e os benefícios dessa abordagem principalmente para o download de novas imagens.
Imagens são Read-Only sempre (apenas para leitura)
Containers representam uma instância de uma imagem
Como imagens são Read-Only os containers criam nova camada (layer) para guardar as alterações


>Fonte: "https://cursos.alura.com.br/course/docker-e-docker-compose"






