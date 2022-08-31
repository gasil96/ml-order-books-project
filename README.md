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















