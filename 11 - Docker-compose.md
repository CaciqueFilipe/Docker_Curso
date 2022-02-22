-------------------------------------------------------------------
#Pegando dados de um banco em outro container
-------------------------------------------------------------------
__Estudaremos uma tecnologia chamada Docker Compose, que nos auxiliará a lidar com múltiplos containers simultaneamente.__

__Na aula anterior, para subir a aplicação alura-books, foi necessário subirmos dois containers, executando os seguintes comandos:__

```
docker run -d --name meu-mongo --network minha-rede mongo
docker run --network minha-rede -d -p 8080:3000 douglasq/alura-books:cap05
```

Isso tudo depois de termos construído pelo menos a imagem **douglasq/alura-books**

##O problema

Esses dois comandos criam dois containers, mas subindo eles desse jeito manual, é muito comum esquecermos de passar alguma flag, ou subir o container na ordem errada, sem a devida rede, ou seja, é um trabalho muito manual e facilmente suscetível a erros, isso com somente dois containers.

Esse modo de subir os containers na mão é bom se quisermos criar um ambiente rapidamente, ou são poucos containers, mas quando a aplicação começa a crescer, temos que digitar muitos comandos.

##Funcionamento das aplicações em geral

Na vida real, sabemos que a aplicação é maior que somente dois containers, geralmente temos dois, três ou mais containers para segurar o tráfego da aplicação, distribuindo a carga. Além disso, temos que colocar todos esses containers para se comunicar com o banco de dados em um outro container, mas quanto maior a aplicação, devemos ter mais de um container para o banco também.

E claro, se temos três aplicações rodando, não podemos ter três endereços diferentes, então nesses casos utilizamos um Load Balancer em um outro container, para fazer a distribuição de carga quando tivermos muitos acessos. Ele recebe as requisições e distribui para uma das aplicações, e ele também é muito utilizado para servir os arquivos estáticos, como imagens, arquivos CSS e JavaScript. Assim, a nossa aplicação controla somente a lógica, as regras de negócio, com os arquivos estáticos ficando a cargo do Load Balancer:

![Load Balancer](https://s3.amazonaws.com/caelum-online-public/646-docker/06/imagens/funcionamento-aplicacoes.png)

Se formos seguir esse diagrama, teríamos que criar cinco containers na mão, e claro, cada container com configurações e flags diferentes, além de termos que nos preocupar com a ordem em que vamos subi-los.

##Docker Compose

Ao invés de subir todos esses containers na mão, o que vamos fazer é utilizar uma tecnologia aliada do Docker, chamada Docker Compose, feito para nos auxiliar a orquestrar melhor múltiplos containers. Ele funciona seguindo um arquivo de texto YAML (extensão .yml), e nele nós descrevemos tudo o que queremos que aconteça para subir a nossa aplicação, todo o nosso processo de build, isto é, subir o banco, os containers das aplicações, etc.

Assim, não precisamos ficar executando muitos comandos no terminal sem necessidade. E esse será o foco desta aula, montar uma aplicação na estrutura descrita anteriormente na imagem, que é uma situação comum no nosso dia-a-dia.

Para começarmos a entender como funciona o __Docker Compose__, primeiramente vamos entender como funciona a aplicação que utilizaremos como base. É uma aplicação bem semelhante à utilizada na aula anterior, com o mesmo servidor, rotas e banco de dados. De novidade, é que agora precisamos criar o **NGINX**, que é mais um container que devemos subir.

Então, ou utilizamos a imagem **nginx**, ou criamos a nossa própria. Como vamos configurar o NGINX para algumas coisas específicas, como lidar com os arquivos estáticos, vamos criar a nossa própria imagem, por isso que na aplicação há o **nginx.dockerfile**:

```
FROM nginx:latest
MAINTAINER Douglas Quintanilha
COPY /public /var/www/public
COPY /docker/config/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80 443
ENTRYPOINT ["nginx"]
# Parametros extras para o entrypoint
CMD ["-g", "daemon off;"]
```

Nesse arquivo, nós utilizamos a última versão disponível da imagem do **nginx** como base, e copiamos o conteúdo da pasta public, que contém os arquivos estáticos, e um arquivo de configuração do NGINX para dentro do container. Além disso, abrimos as portas 80 e 443 e executa o NGINX através do comando __nginx__, passando os parâmetros extras __-g__ e __daemon off__.

Por fim, vamos ver um pouco sobre o arquivo de configuração do NGINX, para entendermos um pouco como o load balancer está funcionando.

No arquivo **nginx.conf**, dentro __server__, está a parte que trata de servir os arquivos estáticos. Na porta 80, no localhost, em **/var/www/public**, ele será responsável por servir as pastas __css, img e js__. E todo resto, que não for esses três locais, ele irá jogar para o **node_upstream**.

No **node_upstream**, é onde ficam as configurações para o NGINX redirecionar as conexões que ele receber para um dos três containers da nossa aplicação. O redirecionamento acontecerá de forma circular, ou seja, a primeira conexão irá para o primeiro container, a segunda irá para o segundo container, a terceira irá para o terceiro container, na quarta, começa tudo de novo, e ela vai para o primeiro container e assim por diante.

**Isso já está tudo pronto, basta baixarmos o código da imagem e da aplicação [aqui](https://s3.amazonaws.com/caelum-online-public/646-docker/06/projetos/alura-docker-cap06.zip).**

Agora, no próximo vídeo, escreveremos o responsável por orquestrar a subida de cada uma dessas partes da nossa aplicação, o **docker-compose.yml**.

Para utilizar o __Docker Compose__, devemos criar o seu arquivo de configuração, o **docker-compose.yml**, na raiz do projeto. Em todo arquivo de Docker Compose, que é uma espécie de receita de bolo para construirmos as diferentes partes da nossa aplicação, a primeira coisa que colocamos nele é a versão do Docker Compose que estamos utilizando:

	version: '3'
	
Estamos utilizando a versão 3 pois é a versão mais recente no momento da criação do treinamento. O YAML lembra um pouco o JSON, mas ao invés de utilizar as chaves para indentar o código, ele utiliza espaços.

Agora, começamos a descrever os nossos serviços, os nossos services:

	version: '3'
	services:
	
Um serviço é uma parte da nossa aplicação. Lembrando do nosso diagrama:

![Load Balancer](https://s3.amazonaws.com/caelum-online-public/646-docker/06/imagens/funcionamento-aplicacoes.png)

Temos NGINX, três Node, e o MongoDB como serviços. Logo, se queremos construir cinco containers, vamos construir cinco serviços, cada um deles com um nome específico.

Então, vamos começar construindo o NGINX, que terá o nome nginx:

```
version: '3'
services:
    nginx:
```

Em cada serviço, devemos dizer como devemos construí-lo, como devemos fazer o seu build:

```
version: '3'
services:
    nginx:
        build:
```

O serviço será construído através de um Dockerfile, então devemos passá-lo onde ele está. E também devemos passar um contexto, para dizermos a partir de onde o Dockerfile deve ser buscado. Como ele será buscado a partir da pasta atual, vamos utilizar o ponto:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
```

Construída a imagem, devemos dar um nome para ela, por exemplo douglasq/nginx:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
```

E quando o Docker Compose criar um container a partir dessa imagem, vamos dizer que o seu nome será nginx:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
```

Sabemos também que o NGINX trabalha com duas portas, a 80 e a 443. Como não estamos trabalhando com HTTPS, vamos utilizar somente a porta 80, e no próprio arquivo, podemos dizer para qual porta da nossa máquina queremos mapear a porta 80 do container. Vamos mapear para a porta de mesmo número da nossa máquina:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
            
```

No YAML, toda vez que colocamos um traço, significa que a propriedade pode receber mais de um item. Agora, para os containers conseguirem se comunicar, eles devem estar na mesma rede, então vamos configurar isso também. Primeiramente, devemos criar a rede, que não é um serviço, então vamos escrever do começo do arquivo, sem as tabulações:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"

networks:

```

O nome da rede será production-network e utilizará o driver bridge:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"

networks: 
    production-network:
        driver: bridge
        
```

Com a rede criada, vamos utilizá-la no serviço:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network

networks: 
    production-network:
        driver: bridge
        
```

Isso é para construir o serviço do NGINX, agora vamos construir o serviço do MongoDB, com o nome mongodb. Como ele será construído a partir da imagem mongo, não vamos utilizar nenhum Dockerfile, logo não utilizamos a propriedade build. Além disso, não podemos nos esquecer de colocá-lo na rede que criamos:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network

    mongodb:
        image: mongo
        networks: 
            - production-network

networks: 
    production-network:
        driver: bridge
        
```

Falta agora criarmos os três serviços em que ficará a nossa aplicação, node1, node2 e node3. Para eles, será semelhante ao NGINX, com Dockerfile alura-books.dockerfile, contexto, rede production-network e porta 3000:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network

    mongodb:
        image: mongo
        networks: 
            - production-network

    node1:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-1
        ports:
            - "3000"
        networks: 
            - production-network

    node2:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-2
        ports:
            - "3000"
        networks: 
            - production-network

    node3:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-3
        ports:
            - "3000"
        networks: 
            - production-network

networks: 
    production-network:
        driver: bridge
        
```

Com isso, a construção dos nossos serviços está finalizada.

##Ordem dos serviços

Por último, quando subimos os containers na mão, temos uma ordem, primeiro devemos subir o mongodb, depois a nossa aplicação, ou seja, node1, node2 e node3 e após tudo isso subimos o nginx. Mas como que fazemos isso no docker-compose.yml?

Nós podemos dizer que os serviços da nossa aplicação dependem que um serviço suba antes deles, o serviço do mongodb:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network

    mongodb:
        image: mongo
        networks: 
            - production-network

    node1:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-1
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node2:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-2
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node3:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-3
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

networks: 
    production-network:
        driver: bridge
        
```

Da mesma forma, dizemos que o serviço do nginx depende dos serviços node1, node2 e node3:

```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network
        depends_on: 
            - "node1"
            - "node2"
            - "node3"

    mongodb:
        image: mongo
        networks: 
            - production-network

    node1:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-1
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node2:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-2
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node3:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-3
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

networks: 
    production-network:
        driver: bridge
        
```

Assim, encerramos a configuração do docker-compose.yml. Vamos ver como subir a aplicação a partir desse arquivo no próximo vídeo.

>Fonte: "https://cursos.alura.com.br/course/docker-e-docker-compose"

