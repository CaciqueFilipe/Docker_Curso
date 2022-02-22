-------------------------------------------------------------------
#Criando um Dockerfile
-------------------------------------------------------------------

Já criamos a imagem, mas por enquanto ela só está na nossa máquina local. Para disponibilizar a imagem para outras pessoas, precisamos enviá-la para o [Docker Hub](https://hub.docker.com/).

O primeiro passo é criar a nossa conta. Com ela criada, no terminal nós executamos o comando docker login e digitamos o nosso login e senha que acabamos de criar.

Após isso, basta executar o comando docker push, passando para ele a imagem que queremos subir, por exemplo:

	docker push douglasq/node
	
Esse comando pode demorar um pouco, mas terminada a sua execução, podemos ver que várias mensagens Mounted from library/node, ou seja, o Docker já sabe que essas camadas podem ser reaproveitadas da imagem do node, então não tem o porquê dessas camadas subirem também, então só as camadas diferentes são enviadas para o Docker Hub.

Mais uma vantagem em se trabalhar com camadas, o Layered File System, pois até na hora de fazer o upload, só é feito das camadas diferentes, as outras são referenciadas da imagem-base que estamos utilizando, no caso a do node.

Por fim, ao acessar a nossa conta do Docker Hub, podemos ver que a imagem está lá. Para baixá-la, podemos utilizar o comando docker pull:

	docker pull douglasq/node
	
Esse comando somente baixa a imagem, sem criar nenhum container acima dela.

Então, esse é um jeito de simples de compartilharmos uma imagem com outras pessoas, através do Docker Hub. A imagem é disponibilizada em um repositório público, mas também podemos disponibilizar em repositórios privados, que no momento da criação do curso, cada usuário pode criar um repositório privado gratuitamente.

--------------------------------------------------
Aprendemos neste capítulo:

A entender o papel do arquivo DockerFile para criar imagens.
O Dockerfile define os comandos para executar instalações complexas e com características específicas.
Vimos os principais comandos como FROM, MAINTAINER, COPY, WORKDIR, RUN, EXPOSE e ENTRYPOINT
A subir uma imagem criada através de um Dockerfile para o Docker Hub e disponibilizar para os desenvolvedores
Lembrando também:

as imagens são read-only sempre
um container é uma instância de uma imagem
para guardar as alterações a docker engine cria uma nova layer em cima da última layer da imagem
Segue também uma breve lista dos comandos utilizados:

	docker build -f Dockerfile - cria uma imagem a partir de um Dockerfile.
	docker build -f CAMINHO_DOCKERFILE/Dockerfile -t NOME_USUARIO/NOME_IMAGEM - constrói e nomeia uma imagem não-oficial informando o caminho para o Dockerfile.
	docker login - inicia o processo de login no Docker Hub.
	docker push NOME_USUARIO/NOME_IMAGEM - envia a imagem criada para o Docker Hub.
	docker pull NOME_USUARIO/NOME_IMAGEM - baixa a imagem desejada do Docker Hub.


>Fonte: "https://cursos.alura.com.br/course/docker-e-docker-compose"

