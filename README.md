<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>
</br>

<h2>Cenario</h2>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades, para isso será utilizado um Cluster de MongoDB no Docker.</p>

<h3>Roteadores:</h3><p>Servidores de Configuração, recebendo a requisição de leitura e escrita e direcionar para a sua partição.</p>
<h3>Shards:</h3><p>Partições de Dados, são os responsaveis por armazenar os dados, cada Shard fica responsavel por um subconjunto de dados no MongoDB.</p>
<h3>ConfigServers:</h3><p>Guardar os metadados das partições (shards).</p>
<p>Para esse projeto será criado um roteador, três ConfigServers e três shards contendo mais dois shards como replicá totalizando 9 Shards no total.</p>


<h2>Criando o Roteador(Cluster)</h2>

```shell
docker network create mongo-vicentin-network
```
<h2>Criando o ConfigServers</h2>
<h3>Criando o Primeiro </h3>

```shell
docker run --name mongo-config-1 --net mongo-vicentin-network -d mongo mongod --configsvr --replSet configserver --port 27018
```

<h3>Criando o Segundo</h3>

```shell
docker run --name mongo-config-2 --net mongo-vicentin-network -d mongo mongod --configsvr --replSet configserver --port 27018
```

<h3>Criando o Terceiro</h3>

```shell
docker run --name mongo-config-3 --net mongo-vicentin-network -d mongo mongod --configsvr --replSet configserver --port 27018
```

<h3>Criando o Quarto</h3>

```shell
docker run --name mongo-config-4 --net mongo-vicentin-network -d mongo mongod --configsvr --replSet configserver --port 27018
```


<p>Apos isso é configurado apenas um dos três containers do ConfigServer, irei usar o Primeiro como exemplo.</p>

```shell
docker exec -it mongo-config-1 mongosh --port 27018
```

```shell
rs.initiate(
   {
      _id: "configserver",
      configsvr: true,
      version: 1,
      members: [
         { _id: 0, host : "mongo-config-1:27018" },
         { _id: 1, host : "mongo-config-2:27018" },
         { _id: 2, host : "mongo-config-3:27018" },
         { _id: 3, host : "mongo-config-4:27018" }
      ]
   }
)
```
<p>Apos a criação dos quatros containers responsaveis pela configuração do servidores, é utilizado o comando "rs.status()" afim de verificar como ficou a configuração dos containers.</p>

```shell
rs.status()
```
<img src="imagem1.png" alt="Imagem_1">

<p>No exemplo acima, o container chamado "mongo-config-1" é o Primario, ou seja. Ele é o principal para fazer os processos dentro do MongoDB, e o restante dos servidores são os Secundarios, ou seja. Caso o "mongo-config-1" sofra por interrupção os algum dos outros três assume. Exemplos a baixo.</p>

<h3>Antes de derrubar: </h3>

<img src="imagem1.png" alt="Imagem_1">
<img src="imagem2.png" alt="Imagem_2">

<h3>Apos derrubar: </h3>

<img src="imagem3.png" alt="Imagem_3">
<img src="imagem4.png" alt="Imagem_4">

<p>A troca de servidores está funcionando, apos um cair e o outro assumir, sendo assim podemos dar continuidade na configuração, agora a criação dos Shards</p>

<h3>Shard 1</h3>

```shell
docker run --name mongo-shard-1a --net mongo-vicentin-network -d mongo mongod --port 27019 --shardsvr --replSet shard1
```

```shell
docker run --name mongo-shard-1b --net mongo-vicentin-network -d mongo mongod --port 27019 --shardsvr --replSet shard1
```

<h3>Shard 2</h3>

```shell
docker run --name mongo-shard-2a --net mongo-vicentin-network -d mongo mongod --port 27020 --shardsvr --replSet shard2
```

```shell
docker run --name mongo-shard-2b --net mongo-vicentin-network -d mongo mongod --port 27020 --shardsvr --replSet shard2
```

<h3>Shard 3</h3>

```shell
docker run --name mongo-shard-3a --net mongo-vicentin-network -d mongo mongod --port 27021 --shardsvr --replSet shard3
```

```shell
docker run --name mongo-shard-3b --net mongo-vicentin-network -d mongo mongod --port 27021 --shardsvr --replSet shard3
```

<h3>Shard 4</h3>

```shell
docker run --name mongo-shard-4a --net mongo-vicentin-network -d mongo mongod --port 27022 --shardsvr --replSet shard4
```

```shell
docker run --name mongo-shard-4b --net mongo-vicentin-network -d mongo mongod --port 27022 --shardsvr --replSet shard4
```

<p>Apos a criação será configurado e iniciado cada um dos shards, para isso é feito uma configuração parecido com a que foi feita nos Servidores, no caso será necessario acessar cada um dos quatro Shards.</p>

<h3>Shard 1</h3>

```shell
docker exec -it mongo-shard-1a mongosh --port 27019
```
```shell
rs.initiate(
   {
      _id: "shard1",
      version: 1,
      members: [
         { _id: 0, host : "mongo-shard-1a:27019" },
         { _id: 1, host : "mongo-shard-1b:27019" },
      ]
   }
)
```
<h3>Shard 2</h3>

```shell
docker exec -it mongo-shard-2a mongosh --port 27020
```
```shell
rs.initiate(
   {
      _id: "shard2",
      version: 1,
      members: [
         { _id: 0, host : "mongo-shard-2a:27020" },
         { _id: 1, host : "mongo-shard-2b:27020" },
      ]
   }
)
```

<h3>Shard 3</h3>

```shell
docker exec -it mongo-shard-3a mongosh --port 27021
```
```shell
rs.initiate(
   {
      _id: "shard3",
      version: 1,
      members: [
         { _id: 0, host : "mongo-shard-3a:27021" },
         { _id: 1, host : "mongo-shard-3b:27021" },
      ]
   }
)
```

<h3>Shard 4</h3>

```shell
docker exec -it mongo-shard-4a mongosh --port 27022
```

```shell
rs.initiate(
   {
      _id: "shard4",
      version: 1,
      members: [
         { _id: 0, host : "mongo-shard-4a:27022" },
         { _id: 1, host : "mongo-shard-4b:27022" },
      ]
   }
)
```

<p>Da mesma forma do servidor, os shards é responsavel por um primario e outro secundario.</p>
<img src="imagem5.png" alt="Imagem_5">
<img src="imagem6.png" alt="Imagem_6">

<p>Agora é so criar o roteador, para que o Shards e os servidores fiquem disponiveis</p>

```shell
docker run -p 27018:27018 --name mongo-router-vicentin --net mongo-vicentin-network -d mongo mongos --port 27018 --configdb configserver/mongo-config-1:27018,mongo-config-2:27018,mongo-config-3:27018,mongo-config-4:27018 --bind_ip_all
```
<p>Por fim, iremos configurar o roteador para que ele conheça os Shards, a configuração é feita pelo seguinte comando.</p>

```shell
docker exec -it mongo-router-vicentin mongosh --port 27018
```
```shell
sh.addShard("shard1/mongo-shard-1a:27019")
sh.addShard("shard1/mongo-shard-1b:27019")
sh.addShard("shard2/mongo-shard-2a:27020")
sh.addShard("shard2/mongo-shard-2b:27020")
sh.addShard("shard3/mongo-shard-3a:27021")
sh.addShard("shard3/mongo-shard-3b:27021")
sh.addShard("shard4/mongo-shard-4a:27022")
sh.addShard("shard4/mongo-shard-4b:27022")
```
<p>Para verificar, se está tudo certo é so usar o comando "sh.status()" </p>

```shell
sh.status()
```
<img src="imagem7.png" alt="Imagem_7">

<p>Agora, o roteador reconhecer os Shard e é possivel usar os serviços</p>

