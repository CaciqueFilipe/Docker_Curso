-------------------------------------------------------------------
# Volumes
-------------------------------------------------------------------

Vimos que os containers nada mais são do que uma pequena camada de leitura e escrita, que funcionam em cima das imagens, que não podem ser modificadas, pois são somente para leitura.

Quando removemos um container (comando docker rm), a camada de leitura e escrita também é removida, o que faz com que os nossos dados também sejam removidos, o que é muito ruim, já que esses dados podem ser importantes, como por exemplo um banco de dados, então toda vez que o container for removido, tudo o que escrevemos nele será jogado fora? Não é isso que queremos, então temos que ver um jeito de persistir esses dados, mas também trabalhando com containers.

É da natureza dos containers a volatilidade, isto é, eles são criados e removidos rapidamente e facilmente, mas devemos ter um lugar para salvar os dados, e esse lugar são os volumes.

## O que são os volumes?
Quando escrevemos em um container, assim que ele for removido, os dados também serão. Mas podemos criar um local especial dentro dele, e especificamos que esse local será o nosso volume de dados.

Quando criamos um volume de dados, o que estamos fazendo é apontá-lo para uma pequena pasta no Docker Host. Então, quando criamos um volume, criamos uma pasta dentro do container, e o que escrevermos dentro dessa pasta na verdade estaremos escrevendo do Docker Host.

Isso faz com que não percamos os nossos dados, pois o container até pode ser removido, mas a pasta no Docker Host ficará intacta.

## Trabalhando com volumes
Sabendo disso, vamos ver como trabalhar com o Docker Host. No Terminal ou PowerShell (ou Docker Quickstart Terminal), criamos um container com o docker run, mas dessa vez utilizando a flag -v para criar um volume, seguido do nome do mesmo:

	docker run -v "/var/www" ubuntu
	
No exemplo acima, criamos o volume /var/www, mas a que pasta no Docker Host ele faz referência? Para descobrir, podemos inspecionar o container, executando o comando docker inspect, passando o seu id para o mesmo:

	docker inspect 8cf7b40ce226
	
Temos uma saída com diversas informações, mas a que nos interessa é o "Mounts":
	
	"Mounts": [
    {
        "Type": "volume",
        "Name": "5e1cbfd48d07284680552e56087c9d5196659600ccd6874bfa3831b51ddd0576",
        "Source": "/var/lib/docker/volumes/5e1cbfd48d07284680552e56087c9d5196659600ccd6874bfa3831b51ddd0576/_data",
        "Destination": "/var/www",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
]

Nele, podemos ver que o /var/www será escrito na nossa máquina no diretório /var/lib/docker/volumes/5e1cbfd48d07284680552e56087c9d5196659600ccd6874bfa3831b51ddd0576/_data, endereço que foi gerado automaticamente pelo Docker. Ou seja, tudo que escrevermos na pasta /var/www do container, na verdade estaremos escrevendo na pasta /var/lib/docker/volumes/5e1cbfd48d07284680552e56087c9d5196659600ccd6874bfa3831b51ddd0576/_data da nossa máquina.

E ao remover o container, a pasta continuará na nossa máquina. Essa pasta gerada pelo Docker pode ser configurada, podemos dizer a pasta que será referenciada pela pasta /var/www do container. Por exemplo, se quisermos escrever dentro do Desktop da nossa máquina, devemos passá-lo antes do volume, separando-os com dois pontos. Além disso, vamos executar o container no modo interativo:

	docker run -it -v "C:\Users\Alura\Desktop:/var/www" ubuntu
	root@abd0286c0083:/#

Ou seja, quando escrevermos na pasta /var/www do container, estaremos escrevendo no Desktop da nossa máquina. Para provar isso, na pasta /var/www, vamos criar um arquivo e escrever nele uma mensagem:

	root@abd0286c0083:/# cd /var/www/
	root@abd0286c0083:/var/www# touch novo-arquivo.txt
	root@abd0286c0083:/var/www# echo "Este arquivo foi criado dentro de um volume" > novo-arquivo.txt

Ao acessarmos o nosso Desktop, o arquivo estará lá, também com a mensagem escrita. E ao remover o container, a sua camada de escrita é removida, mas os arquivos continuam no nosso Desktop.

Então, o uso de volumes é importante para salvarmos os nossos dados fora do container, e esses volumes sempre estarão atrelados ao Docker Host. No caso acima, atrelamos o volume com o Desktop, mas podemos atrelar com um lugar mais seguro, salvando os dados do banco de dados nele, logs, e até mesmo o código fonte, coisa que faremos no próximo vídeo.

--------------------------------------------------------------------
Já vimos que o que escrevemos no volume (pasta /var/www do container) aparece na pasta configurada da nossa máquina local, que no vídeo anterior foi o Desktop. Mas podemos pensar o contrário, ou seja, tudo o que escrevemos no Desktop será acessível na pasta /var/www do container.

Isso nos dá a possibilidade de implementar localmente um código de uma linguagem que não está instalada na nossa máquina, e colocá-lo para compilar e rodar dentro do container. Se o container possui Node, Java, PHP, seja qual for a linguagem, não precisamos tê-los instalados na nossa máquina, nosso ambiente de desenvolvimento pode ser dentro do container.

É isso que faremos, pegaremos um código nosso, que está na nossa máquina, e colocaremos para rodar dentro do container, utilizando essa técnica com volumes.

## Rodando código em um container

Para isso, vamos usar um exemplo escrito Node.js, que pode ser baixado aqui. Até podemos executar esse código na nossa máquina, mas temos que instalar o Node na versão certa em que o desenvolvedor implementou o código.

Agora, como fazemos para criar um container, que irá pegar e rodar esse código Node que está na nossa máquina? Vamos utilizar os volumes. Então, vamos começar a montar o comando.

Primeiramente, como vamos rodar um código em Node.js, precisamos utilizar a sua imagem:

	docker run node
	
Além disso, precisamos criar um volume, que faça referência à pasta do código no nosso Desktop:

	docker run -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" node
	
Agora, para iniciar o seu servidor, executamos o comando npm start. Para executar um comando dentro do container, podemos iniciá-lo no modo interativo ou passá-lo no final do docker run:

	docker run -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" node npm start
	
Por fim, esse servidor roda na porta 3000, então precisamos linkar essa porta a uma porta do nosso computador, no caso a 8080. O comando ficará assim:

	docker run -p 8080:3000 -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" node npm start
	
Executado o comando, recebemos um erro. Nele podemos verificar a seguinte linha:

	npm ERR! enoent ENOENT: no such file or directory, open '/package.json'
	
Isto é, o package.json não foi encontrado, mas ele está dentro da pasta do código. O que acontece é que o container não inicia já dentro da pasta /var/www, e sim em uma pasta determinada pelo próprio container. Por exemplo, se a imagem é baseada no Ubuntu, o container iniciar no root.

Então devemos especificar que o comando npm start deve ser executado dentro da pasta /var/www. Para isso, vamos passar a flag -w (Working Directory), para dizer em qual diretório o comando deve ser executado, a pasta /var/www:

	docker run -p 8080:3000 -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" -w "/var/www" node npm start
	
Agora, ao acessar a porta 8080 no navegador, vemos uma página exibindo a mensagem Eu amo Docker!. E para testar que está mesmo funcionando, podemos editar o arquivo index.html localmente, salvá-lo e ao recarregar a página no navegador, a nova mensagem é exibida! Ou seja, podemos criar um ambiente de desenvolvimento todo baseado em containers, o que ainda facilita o trabalho da nossa equipe, já que se todos utilizarem o container, todos terão o mesmo ambiente de desenvolvimento.

## Melhorando o comando

Por fim, sabemos que podemos executar o Docker de qualquer local da nossa máquina, então podemos executar o comando que fizemos dentro da pasta do nosso projeto. Fazendo isso, podemos melhorar esse comando com o auxílio da interpolação de comandos, já que o comando pwd retorna o nosso diretório atual:

	alura@alura-estudio-03:~$ cd Desktop/volume-exemplo/
	alura@alura-estudio-03:~/Desktop/volume-exemplo$ pwd
	/home/alura/Desktop/volume-exemplo

Assim, ao invés de passar o diretório físico para dentro do comando docker run, podemos utilizar a interpolação de comandos, e interpolar o comando pwd, assim a sua saída será capturada e inserida dentro do docker run:

	alura@alura-estudio-03:~$ cd Desktop/volume-exemplo/
	alura@alura-estudio-03:~/Desktop/volume-exemplo$ pwd
	/home/alura/Desktop/volume-exemplo
	alura@alura-estudio-03:~/Desktop/volume-exemplo$ docker run -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start

Assim, vimos como rodar um código local, que está na nossa máquina, dentro de um container, utilizando a tecnologia dos volumes, linkando a nossa pasta local com uma pasta do container, criando assim um ambiente de desenvolvimento todo baseado em containers.


>Fonte: "https://cursos.alura.com.br/course/docker-e-docker-compose"

