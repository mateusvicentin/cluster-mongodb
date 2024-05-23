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



```shell
docker network create mongo-cluster-0
```
