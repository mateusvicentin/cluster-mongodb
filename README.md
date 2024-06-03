<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/9447150b-41d6-4c7f-8d1e-2596fc1da8f8" alt="apresentação">
</p>

<h1 align="center">Cluster MongoDB utilizando Docker e Monitoramento dos Containers via Zabbix</h1>
<p>Este projeto tem como objetivo aplicar conceitos de dados em um ambiente de banco de dados distribuído, projetando um sistema escalável e eficientemente particionado para lidar com um grande volume de dados. O sistema permite o monitoramento dos clusters presentes no projeto e, caso algum deles pare de funcionar, o Zabbix realizará a notificação.</p>

# Índice 
* [Criando os ConfigServers](#Criando-os-ConfigServers)
* [Criando os Shards](#Criando-os-Shards)
* [Criando o Roteador](#Criando-o-Roteador)
* [Configurando o Zabbix para monitoramento dos Containers](#Configurando-o-Zabbix-para-monitoramento-dos-Containers)
* [Criando o Banco de Dados e Inserindo os Dados](#Criando-o-Banco-de-Dados-e-Inserindo-os-Dados)
* [Adicionando filiais](#Adicionando-filiais)
* [Consulta total de documentos](#Consulta-total-de-documentos)
* [Consultado um ID especifico](#Consultado-um-ID-especifico)
* [Atualização de Inventario](#Atualização-de-Inventario)
* [Excluindo um ID](#Excluindo-um-ID)
* [Testando a Conexão dos Shards e ConfigServers](#Testando-a-Conexão-dos-Shards-e-ConfigServers)
* [Shardizando uma Coleção](#Shardizando-uma-Coleção)
* [Conclusão](#Conclusão)
              
<h2>Cenario:</h2>
<p>O cenário utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermercados com filiais em diferentes cidades. Para isso, será utilizado um Cluster de MongoDB no Docker.</p>

<h3>Roteadores</h3><p>Servidores de configuração, responsáveis por receber requisições de leitura e escrita e direcioná-las para a partição correta.</p>
<h3>Shards</h3><p>Partições de dados, responsáveis por armazenar os dados. Cada shard é responsável por um subconjunto de dados no MongoDB.</p>
<h3>ConfigServers</h3><p>Servidores responsáveis por armazenar os metadados das partições (shards).</p>
<p>Para este projeto, será criado um roteador, três ConfigServers e três shards, cada um contendo mais dois shards como réplica, totalizando nove shards. Foi utilizado um número ímpar para garantir que, em caso de falha de um dos containers, os outros possam assumir.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/e246a7c5-b2c8-4cc6-ba8d-2e761d6db8b9" alt="apresentação">
</p>

<p>Antes de criar o roteador, vamos criar uma rede e colocar todos os containers na mesma rede, para evitar problemas de comunicação entre eles.</p>

```shell
docker network create mongo-vicentin-network-ro
```
<h2>Criando os ConfigServers:</h2>
<h4>ConfigServers</h4>

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

<p>Agora, vamos acessar um dos três ConfigServers e realizar a configuração de inicialização, configurando-os para que se comuniquem entre si, de forma que um deles seja o principal e os outros dois sejam secundários, assumindo o papel de principal caso o principal atual falhe.</p>

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
<p>Nesse caso, o <b>mongo-config1</b> é o principal, e os <b>mongo-config2</b> e <b>mongo-config3</b> são os secundários. Caso o principal perca comunicação ou sofra interrupções, um dos outros dois assumirá o papel de principal.</p>

<h2>Criando os Shards:</h2>
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
<p>Após isso, vamos configurar o shard1, acessando-o de uma forma similar à que acessamos o ConfigServer, mas configurando apenas o shard1, e realizando o mesmo procedimento com os outros dois shards.</p>

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

<p>Após a configuração dos ConfigServers e dos Shards, vamos configurar o Roteador. É através do Roteador que faremos as requisições de leitura e escrita no MongoDB.</p>

<h2>Criando o Roteador:</h2>

```shell
docker run -p 27018:27018 --name mongo-router-vicentin-1 --net mongo-vicentin-network-ro -d mongo mongos --port 27018 --configdb configserver/mongo-config1:27018,mongo-config2:27018,mongo-config3:27018 --bind_ip_all
```
<p>Acessando a configuração do Roteador</p>

```shell
docker exec -it mongo-router-vicentin-1 mongosh --port 27018 
```
<h2>Adicionando os Shards ao roteador:</h2>

```shell
sh.addShard("shard1/mongo-shard-1-a:27019,mongo-shard-1-b:27019,mongo-shard-1-c:27019")
```
```shell
sh.addShard("shard2/mongo-shard-2-a:27020,mongo-shard-2-b:27020,mongo-shard-2-c:27020")
```
```shell
sh.addShard("shard3/mongo-shard-3-a:27021,mongo-shard-3-b:27021,mongo-shard-3-c:27021")
```
<p>Para verificar se está tudo funcionando, é utilizado o comando <code>sh.status()</code></p>

```shell
sh.status()
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/9c183f15-f149-4877-be48-e3431505a9b8" alt="roteador">
</p>
<p>Agora temos um Cluster MongoDB utilizando Docker, com um roteador configurado para direcionar as requisições de leitura e escrita para os shards corretos, além de três ConfigServers configurados para se comunicarem entre si, garantindo a redundância do sistema em caso de falha.</p>

<h2>Configurando o Zabbix para monitoramento dos Containers:</h2>
<p>Neste processo, criaremos os containers responsáveis por iniciar o servidor do Zabbix. Para garantir um funcionamento sem problemas, adicionaremos os containers do Zabbix à rede <b> mongo-vicentin-network-ro</b> para que todos estejam na mesma rede do MongoDB.</p>

<h4>Banco MYSQL Server: </h4>

```shell
docker run --name mysql-server -t -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix_pwd" -e MYSQL_ROOT_PASSWORD="root_pwd" --network=mongo-vicentin-network-ro --restart unless-stopped -d mysql:8.0-oracle --character-set-server=utf8 --collation-server=utf8_bin --default-authentication-plugin=mysql_native_password
```
<h4>Gateway Java: </h4>

```shell
docker run --name zabbix-java-gateway -t --network=mongo-vicentin-network-ro --restart unless-stopped -d zabbix/zabbix-java-gateway:alpine-6.4-latest
```
<h4>Zabbix Server: </h4>

```shell
docker run --name zabbix-server-mysql -t -e DB_SERVER_HOST="mysql-server" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix_pwd" -e MYSQL_ROOT_PASSWORD="root_pwd" -e ZBX_JAVAGATEWAY="zabbix-java-gateway" --network=mongo-vicentin-network-ro -p 10051:10051 --restart unless-stopped -d zabbix/zabbix-server-mysql:alpine-6.4-latest
```
<h4>Servidor Web: </h4>

```shell
docker run --name zabbix-web-nginx-mysql -t -e ZBX_SERVER_HOST="zabbix-server-mysql" -e DB_SERVER_HOST="mysql-server" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix_pwd" -e MYSQL_ROOT_PASSWORD="root_pwd" --network=mongo-vicentin-network -p 8080:8080 --restart unless-stopped -d zabbix/zabbix-web-nginx-mysql:alpine-6.4-latest
```
<p>O acesso ao Zabbix é feito através do seguinte domínio: <a href="http://localhost:8080/zabbix.php?action=dashboard.view">http://localhost:8080/zabbix.php?action=dashboard.view</a></p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/0dc41db4-e450-48e9-b928-2681feb7a840" alt="roteador">
</p>

<p>Ao adicionar um host no Zabbix, é necessário informar o IP do host. Para descobrir o IP dos containers, utilizamos o seguinte comando:</p>

```shell
docker network inspect mongo-vicentin-network-ro
```
<p>Utilizando o <b>mongo-config1</b> como exemplo, verificamos que o IP dele é <b>172.26.0.6</b>.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/d1df5d89-7608-49cf-b734-4e11d955165c" alt="shell">
</p>
<p>Devemos configurar os seguintes atributos:</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/941ad796-d210-40d6-a987-44854181a25c" alt="shell">
</p>
<ul>
<li><strong>Hostname:</strong> Nome que irá aparecer</li>
<li><strong>Template:</strong> Selecionar o template, no caso utilizaremos o ICMP para monitorar o ping desse host</li>
<li><strong>Hostgroups:</strong> Agrupar este host, como servidor, roteador, banco de dados ou outro tipo de host, no caso utilizaremos o Database</li>
<li><strong>Agent:</strong> IP do host a ser adicionado, no caso 172.26.0.6</li>
</ul>

<p>Feito isso, se o container parar no Docker ou tiver algum problema de acesso, ele aparecerá no Zabbix.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/70a6eeac-fa51-4c67-8f3d-0824924d272b" alt="roteador1">
</p>

<p>Agora, adicionaremos os demais containers para monitorar os 9 shards, os 3 ConfigServers e o Roteador, seguindo o mesmo procedimento, alterando apenas o IP.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/977c43e9-9427-4162-a95c-5e898077ecc7" alt="roteador1">
</p>
<p>Desligaremos alguns containers para verificar se o monitoramento está correto.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/442e43cb-86d5-4c4e-bd26-8f343d4e5212" alt="shell">
</p>
<p>O monitoramento está funcionando corretamente. Caso um container perca conexão ou pare de funcionar, será exibido no dashboard do Zabbix.</p>

<h2>Criando o Banco de Dados e Inserindo os Dados:</h2>
<p>Para realizar esse procedimento, foi criado um script em Python que faz a conexão com o banco de dados, cria o database e a collection, e insere dados aleatórios. Para o projeto, foram definidos os seguintes campos: <b>("id_produto", "nome_produto", "preco_compra", "quantidade", "data_entrada", "data_validade")</b>.</p>

<h4>Conexão</h4>

```python
client = MongoClient('localhost', 27018)
db = client.vicentin_matriz
collection = db.produtos_estoque_A
```
<p>O database será denominado <b>vicentin_matriz</b> e a collection <b>produtos_estoque_A</b>.</p>

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
<p>Foi criada uma variável nomes_produtos contendo nomes de produtos encontrados em supermercados. Ao inserir os dados, o script seleciona aleatoriamente um dos nomes informados. Nesta primeira database e collection, inseriremos 50.000 produtos.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/36605196-d8d6-4e30-9cb6-0b37f921db6c" alt="mongodb1">
</p>
<p> Após a inserção de 50.000 produtos, ao conectar ao banco pelo software <b>MongoDB Compass</b> utilizando <b>mongodb://localhost:27018</b>, teremos essa tela ao acessar a database e a collection criada pelo script.</p>
<p>Todo database criado é alocado em um dos três shards de forma aleatória. Como a ideia é que os dados e informações sejam divididos, o próprio roteador decide em qual shard a informação será mantida.</p>

<h4>Verificando em qual Shard o Database está alocado.</h4>
<p>Para verificar, acessamos o roteador e utilizamos novamente o comando <code>sh.status()</code>.</p>

```shell
sh.status()
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/08d09e53-9996-41d8-96eb-33a865e8167f" alt="mongodb1">
</p>

<p>A ideia é lidar com milhares de produtos. Portanto, neste mesmo database e collection, inseriremos mais 3.250.000 produtos aleatoriamente.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/58bf467c-d819-41d4-8009-bf4fcbfd186b" alt="mongodb2">
</p>

<h2>Scripts de consulta de estoque, atualizações de inventário e adição de novas filiais:</h2>
<p>Irei aproveitar a conexão feita anteriormente para adicionar os produtos, então será utilizado o mesmo método.</p>

```python
client = MongoClient('localhost', 27018)
db = client.vicentin_matriz
collection = db.produtos_estoque_A
```
<h2>Adicionando filiais:</b></h2>

<p> Agora, criaremos seis filiais, denominadas <b>vicentin_filial_D, vicentin_filial_E, vicentin_filial_F, vicentin_filial_D, vicentin_filial_E</b> e <b>vicentin_filial_F</b>.

```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_A
collection = db.produtos_estoque_A
```
```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_B
collection = db.produtos_estoque_A
```
```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_C
collection = db.produtos_estoque_A
```
```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_D
collection = db.produtos_estoque_A
```
```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_E
collection = db.produtos_estoque_A
```
```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_F
collection = db.produtos_estoque_A
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/3b54e484-16d2-420a-93a8-67e94d41bced" alt="mongodb3">
</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/e9c2accb-6bbc-4169-9a74-aa8745372c58" alt="mongodb4">
</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/8948ea7d-c86a-4032-be7b-e9bb73c2b4cb" alt="mongodb5">
</p>

<p>Vamos verificar em quais shards os databases <b>vicentin_filial_A, vicentin_filial_B</b> e <b>vicentin_filial_C</b> estão alocados.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/4aaad6cf-2cd8-415c-a4e6-b306bbd82aa2" alt="mongodb7">
</p>

<p>Vamos verificar em quais shards os databases <b>vicentin_filial_D, vicentin_filial_E</b> e <b>vicentin_filial_F</b> estão alocados.</p>

<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/ca2246c3-8895-497e-bc67-9dce5c53424a" alt="mongodb7">
</p>

<h2>Consulta total de documentos:</b></h2>
<p>Foi utilizada a biblioteca time para verificar o tempo que a solicitação de busca no banco demora.</p>

```python
start_time = time.time()
result_count = collection.count_documents({})

end_time = time.time()
execution_time = end_time - start_time

print(f"Consulta de estoque para vicentin_matriz retornou {result_count} produtos.")
print(f"Tempo de execução: {execution_time:.4f} segundos")
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/3c1a08ef-48a7-4a5b-8823-80638bd0dc8e" alt="mongodb7">
</p>
<p>Caso queira testar algum outro banco, troque <code>db = client.vicentin_matriz</code> pelo nome do database desejado e <code>collection = db.produtos_estoque_A</code> pelo nome da collection.</p>
<h4>Exemplo:</h4>

```python
client = MongoClient('localhost', 27018)
db = client.vicentin_filial_A
collection = db.produtos_estoque_A
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/dbba5880-5251-4e37-aeb0-5a7631dddd54" alt="mongodb7">
</p>
<h2>Atualização de Inventario:</b></h2>

```python
produto_ids = collection.distinct("id")
print(f"IDs de produtos disponíveis: {produto_ids[:5]}... (total de {len(produto_ids)})")

product_id_to_update = int(input("Digite o ID do produto que deseja atualizar: "))

if product_id_to_update not in produto_ids:
    print("ID do produto não encontrado.")
else:
    new_quantity = int(input("Digite a nova quantidade para o produto: "))

    start_time = time.time()

    update_result = collection.update_one(
        {"id": product_id_to_update},
        {"$set": {"quantidade": new_quantity, "data_entrada": datetime.now()}}
    )

    end_time = time.time()
    execution_time = end_time - start_time

    print(f"Produto com ID {product_id_to_update} atualizado.")
    print(f"Tempo de execução: {execution_time:.4f} segundos")
```
<p>Irei consultar dentro do MongoDB, para facilitar apenas uma informação. Irei buscar o ID: <b>81063928</b></p>

<h4>Antes:</b></h4>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/2ae4bdbc-e3df-4177-b25f-e60555325f14" alt="mongodb7">
</p>
<h4>Informando o ID para atualizar:</b></h4>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/7ba66f51-55a3-4c87-a354-77c9e483e79e" alt="mongodb7">
</p>
<h4>Depois:</b></h4>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/d9a0ed62-a063-4c77-aa08-629183361153" alt="mongodb7">
</p>

<h2>Consultado um ID especifico:</b></h2>

```python
product_id_to_query = int(input("Digite o ID do produto que deseja consultar: "))
start_time = time.time()

result = collection.find_one({"id": product_id_to_query})

end_time = time.time()
execution_time = end_time - start_time

if result:
    print("Produto encontrado:")
    print(result)
else:
    print("Produto não encontrado.")

print(f"Tempo de execução: {execution_time:.4f} segundos")
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/be398cfe-8692-460e-9c34-1634f29f3de7" alt="mongodb7">
</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/90f9b436-7525-46b6-b029-0af13a7dc3c6" alt="mongodb7">
</p>

<p>Neste exemplo, utilizo o ID <b>45619377</b> para buscar dentro do banco de dados. Caso o ID informado não exista, será retornada uma mensagem de erro. Se qualquer um dos scripts não encontrar o ID, a mensagem será retornada.</p>

<h2>Excluindo um ID:</b></h2>

```python
def excluir_produto_por_id(produto_id):
    start_time = time.time()

    produto = collection.find_one({"id": produto_id})
    
    if produto:
        result = collection.delete_one({"id": produto_id})
        if result.deleted_count > 0:
            print("Produto excluído com sucesso:")
            print(produto)
        else:
            print(f"Falha ao excluir o produto com ID {produto_id}.")
    else:
        print(f"Produto com ID {produto_id} não encontrado.")
    
    end_time = time.time()
    execution_time = end_time - start_time
    print(f"Tempo de execução: {execution_time:.4f} segundos")

product_id_to_delete = int(input("Digite o ID do produto que deseja excluir: "))
excluir_produto_por_id(product_id_to_delete)
```
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/055e40cb-7d92-465c-9dd7-b7ccfa92cf7b" alt="mongodb7">
</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/e74c2155-5d62-4ae0-88f6-065e64c15181" alt="mongodb7">
</p>

<p>No caso, após excluir o ID, ao consultar novamente no MongoDB, ele já não constará mais.</p>

<h2>Testando a Conexão dos Shards e ConfigServers:</h2>
<p>Para isso, irei acessar o status de um dos shards e ConfigServer, para identificar qual é o principal e derrubar essa conexão para verificar se o outro container irá assumir como principal. Irei utilizar o comando <code>rs.status()</code no ConfigServer3</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/2796013f-3e74-4de9-bb94-e59b9fea91bf" alt="mongodb7">
</p>
<p>O mongo-config1 é o principal. Irei encerrar a instância desse cluster no Docker.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/348ede1b-766c-4ffc-aaaf-e91ec28a8928" alt="mongodb7">
</p>
<p>Como podemos ver, o mongo-config1 está offline e, automaticamente, o mongo-config2 se tornou o principal.</p>

<h2>Testando em um dos shards</h2>
<p>Acessando o mongo-shard-3, vemos que o mongo-shard-3-c é o principal. Da mesma forma que no ConfigServer, irei encerrar apenas essa instância.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/951774ef-9c43-472a-8812-070edf114010" alt="mongodb7">
</p>
<p>Apos encerrar apenas essa instância.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/73c3cb6a-4529-48d3-9c1a-78a72bbd26c5" alt="mongodb7">
</p>
<p>Já nesse caso, quem se tornou o principal após desligar o mongo-shard-3-c foi o mongo-shard-3-a.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/05771821-1def-472b-a152-b7aab6221b0f" alt="mongodb7">
</p>
<p>De todo modo, como configuramos o Zabbix para monitorar, ao acessar o dashboard, podemos confirmar que os dois containers não estão funcionando.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/05dc5910-f537-4c52-83bd-6cf96cf057a9" alt="mongodb7">
</p>

<h2>Em casos futuros, adição de novos Shards e ConfigServers:</h2>
<p>Conforme o banco for crescendo, é aconselhável adicionar novos shards e config servers para que as configurações e os dados presentes no MongoDB fiquem sempre bem distribuídos entre eles, evitando demora nas consultas ou sobrecarga em algum dos servidores. Isso se dá conta, pois após inserir mais dados na matriz e fazer uma consulta, o tempo de resposta está um pouco mais elevado.</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/8d9a031a-dc1e-4ba1-ae7c-b6d1122db433" alt="mongodb7">
</p>

<p>Caso fosse adicionado mais um shard e um config server, a representação do projeto ficaria assim:</p>
<p align="center">
  <img src="https://github.com/mateusvicentin/cluster-mongodb/assets/31457038/9c2841d3-63de-4c99-83ab-7e324a2ad573" alt="mongodb7">
</p>

<h2>Shardizando uma Coleção</h2>
<p>Vamos configurar o particionamento do banco de dados, para que possamos manipular e armazenar grandes volumes de dados, distribuindo a carga de trabalho de forma eficiente.</p>
<p>Para este procedimento, utilizaremos o comando <code>sh.enableSharding("nomedobanco")</code>. Este comando deve ser usado para todos os bancos de dados criados.</p>

```python
sh.enableSharding("vicentin_matriz")
sh.enableSharding("vicentin_filial_A")
sh.enableSharding("vicentin_filial_B")
sh.enableSharding("vicentin_filial_C")
sh.enableSharding("vicentin_filial_D")
sh.enableSharding("vicentin_filial_E")
sh.enableSharding("vicentin_filial_F")
```
<h4>Definir a Chave de Shard na Coleção</h4>
<p>Vamos criar primeiro um índice para ser usado como chave do shard com o comando <code>db.myCollection.createIndex({ shardKeyField: 1 })</code>. Este índice garante que o MongoDB possa distribuir os documentos da coleção entre os shards.</p>
<p>Vamos criar um índice para cada banco e cada coleção, como mostrado no exemplo abaixo.</p>

```python
db.produtos_estoque_A.createIndex({ shardKeyField: 1 })
```
<p>Após habilitar o sharding em cada banco e criar os índices para eles, vamos distribuir os documentos de uma coleção entre os shards. Para isso, vamos compartilhar cada banco e coleção, permitindo que o MongoDB divida a coleção em pedaços (chunks) e distribua esses pedaços entre os shards.</p>
<p>Para este procedimento, utilizaremos o comando <code>sh.shardCollection("nomebanco.nomecolecao", { shardKeyField: 1 })</code>.</p>

```python
sh.shardCollection("vicentin_matriz.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_A.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_B.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_C.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_D.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_E.produtos_estoque_A", {shardKeyField: 1})
sh.shardCollection("vicentin_filial_F.produtos_estoque_A", {shardKeyField: 1})
```
<h2>Conclusão:</b></h2>
<p>Chegamos ao final do projeto, onde foi criado um cluster de MongoDB composto por 3 ConfigServers, 3 shards com 3 nós cada, e 1 roteador. Todo o monitoramento dos 13 hosts é realizado via Zabbix, utilizando os IPs distribuídos pelo próprio Docker. O cluster é autossuficiente para consultas e operações de 7 bancos de dados, cada um contendo milhões de dados armazenados.

As adições de filiais e de dados foram realizadas através de um script em Python, que também incluiu scripts para consultas de informações, atualizações de inventário e exclusão de itens. Testamos a funcionalidade de failover, verificando que, caso um servidor principal seja desligado, um dos secundários assume sua função.

Como atualização futura, planejamos adicionar o plugin do Grafana para gerar gráficos e criar dashboards de monitoramento dos clusters, permitindo a análise de CPU, memória e outras métricas coletadas pelo Zabbix. Além disso, caso as consultas comecem a ficar lentas ou surjam outros problemas de desempenho, será possível adicionar mais shards ou ConfigServers para balancear os dados novamente.</p>


