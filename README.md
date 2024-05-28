<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>
</br>

<h2>Cenario</h2>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades, para isso será utilizado um Cluster de MongoDB no Docker.</p>

<h3>Roteadores</h3><p>Servidores de Configuração, recebendo a requisição de leitura e escrita e direcionar para a sua partição.</p>
<h3>Shards</h3><p>Partições de Dados, são os responsaveis por armazenar os dados, cada Shard fica responsavel por um subconjunto de dados no MongoDB.</p>
<h3>ConfigServers</h3><p>Guardar os metadados das partições (shards).</p>
<p>Para esse projeto será criado um roteador, três ConfigServers e três shards contendo mais dois shards como replicá totalizando 9 Shards no total. Foi utilizado o numero impar, pois caso tenha falhas ou problemas com algum dos containers o outro consiga assumir.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/e246a7c5-b2c8-4cc6-ba8d-2e761d6db8b9" alt="apresentação">
</p>

<p>Antes de criar o roteador, vamos criar uma rede e colocar todos os containers na mesma rede, para que não ocorra nenhum problema de comunicação entre eles.</p>

```shell
docker network create mongo-vicentin-network-ro
```
<h2>Criando os ConfigServers</h2>
<h4>ConfigServer1</h4>

```shell
docker run --name mongo-config1 --net mongo-vicentin-network-ro -d mongo mongod --configsvr --replSet configserver --port 27018
```
<h4>ConfigServer2</h4>

```shell
docker run --name mongo-config2 --net mongo-vicentin-network-ro -d mongo mongod --configsvr --replSet configserver --port 27018
```
<h4>ConfigServer3</h4>

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
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/d0050455-3383-4227-ac52-922bc7c0dd3d" alt="configserver">
</p>
<p>Nesse caso o "mongo-config1" é o principal, o "mongo-config2 e mongo-config3" são os secundarios, caso o principal perca comunicação ou sofra interrupções, um dos outros dois irá assumir e virar o principal.</p>

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
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/716bf6d7-e6eb-412a-9730-f7cfc5b5da07" alt="shard1">
</p>
<p align="center"> Nesse caso do Shard1, o "mongo-shard-1-b" é o principal, então o "mongo-shard-1-a e mongo-shard-1-c" são os secundarios.</p>

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
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/060e9ab4-5721-4f9d-adf9-64a75ccb64d0" alt="shard2">
</p>
<p align="center"> Nesse caso do Shard2, o "mongo-shard-2-b" é o principal, então o "mongo-shard-2-a e mongo-shard-2-c" são os secundarios.</p>

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
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/9d913fe7-3323-480e-b493-78ac4af594e3" alt="shard3">
</p>
<p align="center"> Nesse caso do Shard3, o "mongo-shard-3-b" é o principal, então o "mongo-shard-3-a e mongo-shard-3-c" são os secundarios.</p>

<p>Apos criar e configurar todos os ConfigServers e fazer o mesmo com o Shard, iremos criar o Routeador e associar todos os ConfigServers a ele, e acessar a configurar dele e associar os Shards a ele tambem, da mesma forma que configuramos os Shards e o ConfigServer</p>

<h2>Criando o Roteador</h2>

```shell
docker run -p 27018:27018 --name mongo-router-vicentin-1 --net mongo-vicentin-network-ro -d mongo mongos --port 27018 --configdb configserver/mongo-config1:27018,mongo-config2:27018,mongo-config3:27018 --bind_ip_all
```
<p>Acessando a configuração do Roteador</p>

```shell
docker exec -it mongo-router-vicentin-1 mongosh --port 27018 
```
<h2>Adicionando os Shards ao roteador</h2>

```shell
sh.addShard("shard1/mongo-shard-1-a:27019,mongo-shard-1-b:27019,mongo-shard-1-c:27019")
```
```shell
sh.addShard("shard2/mongo-shard-2-a:27020,mongo-shard-2-b:27020,mongo-shard-2-c:27020")
```
```shell
sh.addShard("shard3/mongo-shard-3-a:27021,mongo-shard-3-b:27021,mongo-shard-3-c:27021")
```
<p>Para verificar se está tudo funcionando, é utilizado o comando "sh.status()"</p>

```shell
sh.status()
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/9c183f15-f149-4877-be48-e3431505a9b8" alt="roteador">
</p>
<p align="center"> Teremos uma tela parecida com essa, os shards separados cada um com seu conjunto de servidores.</p>

<h2>Criando o Banco de Dados e Inserindo os Dados</h2>
<p>Para realizar esse procedimento, foi criado um script em Python que faz a conexão com o banco e cria o database e a collection com a inserção de dados aleatorios, para o projeto foi criado os seguintes dados. ("id_produto", "nome_produto", "preco_compra", "quantidade", "data_entrada", "data_validade")</p>

<h4>Conexão</h4>

```python
client = MongoClient('localhost', 27018)
db = client.vicentin_matriz
collection = db.produtos_estoque_A
```
<p>O database irá chamar "vicentin_matriz" e a collection "produtos_estoque_A"</p>

<h4>Gerar Dados</h4>

```python
def gerar_produto_aleatorio():
    id_produto = random.randint(0, 99999999)
    nome_produto = random.choice(nomes_produtos)
    preco_compra = round(random.uniform(1.0, 80.0), 2)
    quantidade = random.randint(1, 100)
    data_entrada = datetime.now() - timedelta(days=random.randint(1, 365))
    data_validade = data_entrada + timedelta(days=random.randint(1, 365))  # Data de validade após a entrada
    return {
        "id": id_produto,
        "nome": nome_produto,
        "preco_compra": preco_compra,
        "quantidade": quantidade,
        "data_entrada": data_entrada,
        "data_validade": data_validade
    }
```
<p>Foi criado uma variavel "nomes_produtos" contendo nomes de produtos encontrados em supermercados, e quando forem inseridos o script irá escolher um dos nomes informados para inserir no banco.Nessa primeira database e collection, irei inserir 50,000 produtos</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/36605196-d8d6-4e30-9cb6-0b37f921db6c" alt="mongodb1">
</p>
<p align="center"> Apos a inserção de 50000 produtos, ao conectarmos ao banco pelo Software do "MongoDB Compass" utilizando o "mongodb://localhost:27018" teremos essa tela ao acessar a database e o collection criado a partir do script.</p>

<p>Todo database criado é alocado em um dos três shards de forma aleatoria, como a ideia é que seja divido os dados e as informações, o proprio roteador decide em qual deles será mantido a informação.</p>

<h4>Verificando em qual Shard o Database está alocado.</h4>
<p>Para verificar, acessamos o roteador e utilizamos novamente do comando "sh.status()"</p>

```shell
sh.status()
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/08d09e53-9996-41d8-96eb-33a865e8167f" alt="mongodb1">
</p>
<p align="center"> Nesse caso o seu Shard principal é o Shard3, porem todos os outros Shards conseguem chegar até essas informações pois todos estão na mesma rede e estão configurados dentro do roteador para serem visto.</p>

<p>A ideia é ele seja capaz de lidar com milhares de produtos, então nesse mesmo database e collection irei inserir mais 3,250,000 produtos de forma aleatoria.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/58bf467c-d819-41d4-8009-bf4fcbfd186b" alt="mongodb2">
</p>
<p align="center">O databse apos a inserção de mais 3,250,000 produtos.</p>




