<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>
</br>

<h2>Cenario</h2>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades.</p>
<p>Para isso será utilizado três tipos de serviços diferentes, que são:</p>
</br>

<h3>Roteadores:</h3><p>Responsáveis pelas requisições de escrita e leitura.</p>
<h3>Shards:</h3><p>Guardar os objetos no banco de dados.</p>
<h3>ConfigServers:</h3><p>Guardar os metadados do nosso cluster.</p>



<h2>Criando o Roteador(Cluster)</h2>

```shell
docker network create mongo-cluster-0
```
<h2>Criando o ConfigServers</h2>
<h3>Criando o Primeiro </h3>

```shell
docker run --name mongo-config-1 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017
```

<h3>Criando o Segundo</h3>

```shell
docker run --name mongo-config-2 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017
```

<h3>Criando o Terceiro</h3>

```shell
docker run --name mongo-config-3 --net mongo-cluster-0 -d mongo mongod --configsvr --replSet config-se
rvers --port 27017
```

<p>Apos isso é configurado apenas um dos três containers do ConfigServer, irei usar o Primeiro como exemplo.</p>

```shell
 docker exec -it mongo-config-1 mongosh
```

```shell
rs.initiate( 
{
   _id: "config-servers", 
   configsvr: true, 
   version: 1,
   members: [ 
    { _id: 0, host: "mongo-config-1:27017" },
    { _id: 1, host: "mongo-config-2:27017" }, 
    { _id: 2, host: "mongo-config-3:27017" }
]
}
)
```
