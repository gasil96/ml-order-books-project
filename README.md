# Projeto Livro de Ofertas
Este repositório contem em sua raiz o link para outros repositórios assim como o docker-compose com todos os seus containers orquestrados.

**TL;DR:** Este projeto se baseia em um [order book](https://blog.mercadobitcoin.com.br/order-book-veja-como-funciona-e-realize-bons-investimentos)(livro de ofertas), um usuário podera se cadastrar e gerar uma carteira, logo em seguida pode publicar no livro de ofertas ordens de compra ou venda de uma determinada quantidade de ativo.
Assim que uma ordem de compra se equaliza a uma ordem de venda a transação é processada e ambos os usuários tem os valores e quantidades refletidos em suas respectivas carteiras.

## Stacks empregadas no projeto
* Java - 11
* Maven - 3.8.1
* Spring Boot - 2.6.1
* RabbitMQ - 3.10.7
* MongoDB - 6.0
* MySQL - 5.7
* Prometheus - 2.37
* Grafana - 6.3.0
* Nginx - 1.19
* Docker - 20.20.12
* Docker Compose - 2.2.3

## Arquitetura
<p align="center">
  <img src="https://user-images.githubusercontent.com/48265863/187752963-2a5bdf77-fa2e-44db-beab-ee2d07e199b7.png">
</p>

Como solução de arquitetura a aplicação foi dividia em três microsserviços no qual dois deles foram implementados.
* **ml-orde** API CORE é onde ocorre todo registro de novas ordens de compra e/ou venda e reponsável por executar o processamento das operações nas carteiras
* **ml-registe** API de Registro recebe de forma assíncrona os registros de movimentações e armazena em um banco de dados.
* **Exchange RabbitMQ** Movimenta numa fila as transações da API Core para a API de Registro
* **ml-wallet(não implementada)** Responsável por gerênciar as contas de usuários e suas respectivas carteiras (recebendo informações atráves de uma exchange tambem seria responsável por executar a atualização do saldo)
* **WAF** Web Application Firewall adiciona uma camada de segurança ao receber as multiplas requisições de usuário.
* **NGINX** Localmente funcionará como *Load Balance* em *Round Robin* quando subir a api core *escalada* (idealmente seria o ELB da amazon em um ambiente produtivo)
* **Prometheus** Monitora informações de métricas do actuator e repassa ao Grafana
* **Grafana** Expoem infomações de métricas da aplicação em dashboards.
* **MongoDB** Banco de dados não relacional
* **MySQL&&** Banco de dados relacional

## Banco de Dados

Para escolha do banco de dados foi utilizado o **Teorema de CAP** como príncipio e tomada de decisão.
<p align="center">
  <img src="https://user-images.githubusercontent.com/48265863/187760630-749b77b7-72ea-4785-919c-30552baf1562.png">
</p>

* MySQL - Utilizado em dois databases diferentes sendo o principal usado para armazenar as infomrações de usuário e carteira e o secundário informações de registro
* MongoDB - Utilizado para armazenar os dados de transações.
<p align="center">
  <img src="https://user-images.githubusercontent.com/48265863/187760211-245ce631-3abf-44b0-8a3d-f2d57aafd756.png">
</p>

## ML-ORDER (CORE)

Aplicação principal, possui suas métricas monitoradas e nesse exemplo também carrega a responsabilidade de gerenciamento de usuários e suas carteiras 

![image](https://user-images.githubusercontent.com/48265863/187761351-87ad80c2-905d-4912-a06c-a98572d0bc27.png)

## ML-REGISTER

Aplicação que recebe e armazena informações de registro, tambem contem um endpoint para listagem do que foi armazenado.

![image](https://user-images.githubusercontent.com/48265863/187761635-510bca0d-acee-46e5-bcb1-5b2aed994906.png)

OBS: Ambas as API se comunicam somente atráves da exchange no RabbitMQ

## Observabilidade

Ambas as API's possuem logs padronizados com SL4J, cada service, controller, ou métodos pertinentes possuem logs de entrada, saída e erro.
Juntamente com isso temos o Spring Actuator informando a saúde da aplicação e determinadas métricas. Tambem usamos o Prometheus e Grafana como mencionado anteriormente

## Portas utilizadas e algumas observações

Porta       | Porta Utilizada
---------   | --------
nginx       | http:localhost:4000
ml-order    | http:localhost:8080
ml-register | http:localhost:8081
rabbitmq    | http:localhost:15672
grafana     | http:localhost:3000
prometheus  | http:localhost:9090
mysql       | http:localhost:3306
mongo       | http:localhost:27017

Obs¹: Não acesse o ml-order atáves a porta 8080, ela esta exposta apenas para outros serviços consumirem, o responsavel por acessar essa aplicação é o NGinx na porta 4000

Obs²: Após a súbida é necessário apontar o datasource do prometheus no grafana para que ele possa mostrar as métricas corretamente, e alguns ids de dashborad são:
*4701 Para o prometheus
*14430 Para métricas do Java/Spring, como Threed, Max Pool, etc...

## Como rodar o projeto na sua máquina | How To

Requisitos mínimos
* Java 11
* Maven
* Docker
* Docker Compose

1 - Baixe esse repositorio a partir da branch main 
ˋˋˋ
git clone <esse repositório>
 ˋˋˋ
2 - Navege para os diretórios de cada aplicação e rode o seguinte comando maven
ˋˋˋ
mvn clean install
 ˋˋˋ
3 - Retorne para o diretório raiz deste repositório e rode o seguinte comando docker:
ˋˋˋ
docker-compose up
 ˋˋˋ
4 - Aguarda a subida completa e pronto, você ja pode navegar entre as ferrametas, documentações e testar as aplicações

dica: na raiz desse repositorio existe um arquivo chamado 'order-books-project.postman_collection.json' com todos os endpoints ja mapeados com exemplos... pronto para ser importado no postman =) 
