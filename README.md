<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>
</br>

<h2>Cenario</h2>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades, para isso será utilizado um Cluster de MongoDB no Docker.</p>

<h3>Roteadores:</h3><p>Servidores de Configuração, recebendo a requisição de leitura e escrita e direcionar para a sua partição.</p>
<h3>Shards:</h3><p>Partições de Dados, são os responsaveis por armazenar os dados, cada Shard fica responsavel por um subconjunto de dados no MongoDB.</p>
<h3>ConfigServers:</h3><p>Guardar os metadados das partições (shards).</p>
<p>Para esse projeto será criado um roteador, três ConfigServers e três shards contendo mais dois shards como replicá totalizando 9 Shards no total.</p>

<img src="imagem1.png" alt="Imagem_7">

<p>Antes de criar o roteador, vamos criar uma rede e colocar todos os containers na mesma rede, para que não ocorra nenhum problema de comunicação entre eles.</p>

```shell
docker network create mongo-vicentin-network-ro
```
<h2>Criando os ConfigServers</h2>
<h4>ConfigServer1</h4>

```shell
docker run --name mongo-config1 --net mongo-vicentin-network-ro -d mongo mongod --configsvr --replSet configserver --port 27018
```
<h4>ConfigServer1</h4>

```shell
docker run --name mongo-config2 --net mongo-vicentin-network-ro -d mongo mongod --configsvr --replSet configserver --port 27018
```
<h4>ConfigServer1</h4>

```shell
docker run --name mongo-config3 --net mongo-vicentin-network-ro -d mongo mongod --configsvr --replSet configserver --port 27018
```

<p>Agora, vamos acessar um dos três ConfigServers e realizar a configuração de inicialização, configurando para que os três se comuniquem para que um deles seja o Principal e os outros dois seja o Secundario, caso tenha algum problema com o Principal.</p>

```shell
docker exec -it mongo-config1 mongosh --port 27018
```
```shell
rs.initiate({
   _id: "configserver",
   configsvr: true,
   version: 1,
   members: [
      { _id: 0, host: "mongo-config1:27018" },
      { _id: 1, host: "mongo-config2:27018" },
      { _id: 2, host: "mongo-config3:27018" }
   ]
})
```
<h2>Criando os Shards</h2>
<h4>Shard1</h4>

```shell
docker run --name mongo-shard-1-a --net mongo-vicentin-network-ro -d mongo mongod --port 27019 --shardsvr --replSet shard1
```
```shell
docker run --name mongo-shard-1-b --net mongo-vicentin-network-ro -d mongo mongod --port 27019 --shardsvr --replSet shard1
```
```shell
docker run --name mongo-shard-1-c --net mongo-vicentin-network-ro -d mongo mongod --port 27019 --shardsvr --replSet shard1
```
<p>Apos isso vamos configurar o Shard1, acessando de uma forma parecida com a que a gente acessou o ConfigServer, porem dessa vez configurado apenas o Shard1, e fazendo o mesmo com os outros dois shards</p>

```shell
docker exec -it mongo-shard-1-a mongosh --port 27019 
```
```shell
rs.initiate({
   _id: "shard1",
   version: 1,
   members: [
      { _id: 0, host: "mongo-shard-1-a:27019" },
      { _id: 1, host: "mongo-shard-1-b:27019" },
      { _id: 2, host: "mongo-shard-1-c:27019" }
   ]
})
```
<h4>Shard2</h4>

```shell
docker run --name mongo-shard-2-a --net mongo-vicentin-network-ro -d mongo mongod --port 27020 --shardsvr --replSet shard2
```
```shell
docker run --name mongo-shard-2-b --net mongo-vicentin-network-ro -d mongo mongod --port 27020 --shardsvr --replSet shard2
```
```shell
docker run --name mongo-shard-2-c --net mongo-vicentin-network-ro -d mongo mongod --port 27020 --shardsvr --replSet shard2
```
<p>Acessando a configurção do Shard2</p>

```shell
docker exec -it mongo-shard-2-a mongosh --port 27020
```
```shell
docker exec -it mongo-shard-2-a mongosh --port 27020 
rs.initiate({
   _id: "shard2",
   version: 1,
   members: [
      { _id: 0, host: "mongo-shard-2-a:27020" },
      { _id: 1, host: "mongo-shard-2-b:27020" },
      { _id: 2, host: "mongo-shard-2-c:27020" }
   ]
})
```
<h4>Shard3</h4>

```shell
docker run --name mongo-shard-3-a --net mongo-vicentin-network-ro -d mongo mongod --port 27021 --shardsvr --replSet shard3
```
```shell
docker run --name mongo-shard-3-b --net mongo-vicentin-network-ro -d mongo mongod --port 27021 --shardsvr --replSet shard3
```
```shell
docker run --name mongo-shard-3-c --net mongo-vicentin-network-ro -d mongo mongod --port 27021 --shardsvr --replSet shard3
```
<p>Acessando a configurção do Shard3</p>

```shell
docker exec -it mongo-shard-3-a mongosh --port 27021 
```
```shell
docker exec -it mongo-shard-3-a mongosh --port 27021 
rs.initiate({
   _id: "shard3",
   version: 1,
   members: [
      { _id: 0, host: "mongo-shard-3-a:27021" },
      { _id: 1, host: "mongo-shard-3-b:27021" },
      { _id: 2, host: "mongo-shard-3-c:27021" }
   ]
}) 
```

