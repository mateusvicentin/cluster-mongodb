<h1 align="center">Cluster MongoDB utilizando o Docker</h1>
<p align="center">Projeto tem como o objetivo aplicar conceitos de dados em um amiente de banco de dados distruido. Projentando um sistema escalável e eficientemente particionado para lidar com um grade volume de dados.</p>


<h3>Cenario:</h3>
<p>O cenario utilizado é um sistema de gerenciamento de estoque para uma cadeia de supermarcados que possui filiais em diferentes cidades.</p>
<p>Para isso será utilizado três tipos de serviços diferentes, que são:.</p>

<b>Roteadores:</b><p>Responsáveis pelas requisições de escrita e leitura.</p>
<h4>Shards:</h4><p>Guardar os objetos no banco de dados.</p>
<h4>ConfigServers:</h4><p>Guardar os metadados do nosso cluster.</p>
