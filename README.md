<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>
</br>

<h2>Cenario</h2>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades.</p>
<p>Para isso será utilizado três tipos de serviços diferentes, que são:.</p>
</br>

<h4>Roteadores:</h4><p>Responsáveis pelas requisições de escrita e leitura.</p>
<h4>Shards:</h4><p>Guardar os objetos no banco de dados.</p>
<h4>ConfigServers:</h4><p>Guardar os metadados do nosso cluster.</p>



<h2>Criando o Roteador(Cluster)</h2>

```shell
docker network create mongo-cluster-0
```
<h2>Criando o ConfigServers</h2>
<h3>Criando o Primeiro </h3>
docker run --name mongo-config-1 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017

<h3>Criando o Segundo</h3>
docker run --name mongo-config-2 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017

<h3>Criando o Terceiro</h3>
docker run --name mongo-config-3 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017
