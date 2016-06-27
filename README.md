# WiTIX
Witix é uma plataforma de monitoramento, que utiliza um banco de dados nosql [Elasticserach](http://www.elasticsearch.org) e descoberta de dados e insights atraves da ferramenta [Kibana](http://www.elasticsearch.org/overview/kibana/). A figura abaixo ilustra a solução:

![WiTIX Big Piucture](https://docs.google.com/drawings/d/166QS8ElSPxsFoglBsdH544QX_Umvyih4y_ZXaq4Uxyw/pub?w=939&h=517)

A seguir ilustramos alguns exemplos de dashboards que são gerados utilizando a configuração padrão da ferramentao. O dashboard a seguir ilustra, *em tempo real* os servições mais utilizados, tempo de resposta, quantidade de requisições durante o período de tempo informado. 
![Request](https://docs.google.com/drawings/d/1A4wDuMpSF_YQ4_JYiUbwpUYuMYGIjulgDhbMFctfDVM/pub?w=797&h=387)

A seguir um dashboard que permite agrupar os comandos SQL mais lentos.
![SQL](https://docs.google.com/drawings/d/1ehOdKfW89PSGDGtaW8j2_Ftz70Md7lfI5aYRSXXUuVc/pub?w=959&h=518)

É possível também enviar dados de negócio de cruzar esses daados com informações técnica. 
![Indicadores Negocio](https://docs.google.com/drawings/d/11RZ9Of_9J7ht7DJdP3pauQls72A-pxT2QS_qyaRMH1Q/pub?w=801&h=454)

## Ambiente
 - Git
 - [Docker](https://docs.docker.com/engine/installation/)
 
# Executando

## Docker Compose
Utilizando o docker

```shell
git clone https://mlacerda_cit@bitbucket.org/ciandt_it/cit-witix.git
cd witix-docker
docker-compose up 
```

O comendo **docker-compose up** já inicia os quatro containers configuradas no arquivo docker-compose.yml fazendo os links necessários. Importante observar que primeira execução será mais demorada, pois as imagens serão construídas (build) e as dependencias baixadas (kibana, elasticsearch, logstash, debian) do repositório central [docker hub](http://hub.docker.com)

Utilize as URLs abaixo para acessar os serviços: 

* http://localhost:9200/_plugin/head
* http://localhost:5601

Como enviar dados para o elasticsearch? Veja alguns exemplos em [Samples](witix-samples/readme.md)

## Elasticsearch 
O comando abaixo constroi e executa o elasticsearch individualmente. 

```shell
cd elasticsearch
docker build -t witixdocker_witix-elasticsearch
# expose on port 9200. Param --name is used to link docker with another images
docker run -p 9200:9200 --name=witix-elasticsearch  witixdocker_witix-elasticsearch
```

## ElastAlert

Os comandos abaixo fazem o build da imagem docker para o elastalert
```shell
cd elastalert
docker build -t witixdocker_witix-elastalert . 
docker run --link witix-elasticsearch:witix-elasticsearch witixdocker_witix-elastalert
```

## Kibana
O comando abaixo constroi e executa o elasticsearch individualmente. 

```shell
cd kibana
docker build -t witixdocker_witix-kibana
# expose on port 5601. Param link is used to link elasticsearch host
docker run -p 5601:5601 --name=witix-kibana --link witix-elasticsearch:witix-elasticsearch witixdocker_witix-kibana
```

## Petclinic
O comando abaixo, executa aplicação demo petclinic, já instrumentada com os plugins do witix. 

```shell
cd witix-samples/witix-spring-petclinic/
# Require JDK 1.7 and maven 3.3.3 or superior 
mvn tomcat7:run -Dmetrics.enabled=true -Dwitix.elasticsearch.url=http://[Elasticsearch Host]<:[port]> -Dwitix.applicationName=PETCLINIC -Dwitix.requestmonitor.http.collectHeaders=true -Dwitix.web.paths.excluded="/javax.faces.resource, /resources"
```