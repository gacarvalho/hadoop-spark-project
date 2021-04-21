## 📌 PROPOSTA DO PROJETO

O objetivo do projeto é recuperar um arquivo sobre jogos que está localizado no HDFS com Apache Spark e realizar consultas com o SparkSQL. O arquivo 'DADOS_GAME' foi deslocado da máquina local para ficar disponível no HDFS (Hadoop Distributed File System): Você pode consultar como foi o processo neste repositório - https://github.com/gacarvalho/hadoop-hdfs-project 

📢  ETAPA 1: SERVIÇOS DO HADOOP

O primeiro passo é ativar e verificar como está os serviços do Hadoop! Os serviços necessário são: NodeManager, ResourceManager, DataNode, Jps, SecondaryNameNode, NameNode. Você pode analisar na imagem abaixo que todos os serviços estão ativos.

[Imagem 1]

📢  ETAPA 2: CONSULTANDO O ARQUIVO 'DADOS_GAME' NO HDFS

O segundo passo antes de começarmos a utilizar o Spark é consultar o conteúdo do arquivo DADOS_GAME no HDFS (Hadoop Distributed File System). Como você pode observar, foi necessário listar os conteúdos pelo o comando:

```bash
user@user:/usr/local/hadoop$ bin/hdfs dfs -ls /user/igti/DADOS_GAME
```
Logo após foi necessário apresentar o conteúdo do arquivo distribuido que está localizado no HDFS! Vale lembrar que o arquivo não foi divido em mais parte por conta do seu tamanho. O número de registro é de 231! 

```bash
user@user:/usr/local/hadoop$ bin/hdfs dfs -cat /user/igti/DADOS_GAME/part-m-00000
```
[Imagem 2]
[Imagem 2 - 1]

📢  ETAPA 3: COLOCANDO O APACHE SPARK NO MODO ON

Antes de mais nada é necessário se deslocar da pasta do hadoop e ir até a pasta de instalação do Spark e ativar atraves do comando:

```bash
user@user:/usr/local/spark$ bin/spark-shell
```
[Imagem 3]

📢  ETAPA 4: CRIANDO UM RDD 

Para quem tem conhecimento em Spark, sabe que existe duas formas de criar um RDD (Coleção de Dados Imutaveis): (1) Coletando os dados de um sistema de armazenamento externo (2) Aplicando manualmente os valores. Para esse projeto optamos por coletar os dados de um sistema de armazenamento externo, que é o HDFS. Para isso vamos aplicar o caminho do arquivo na ```val dados``` e logo após, vamos contar quantos registros existem no documento:

```bash
scala>  val dados = sc.textFile("hdfs://localhost:54310/user/igti/DADOS_GAME/part-m-00000")
```
[Imagem 4]

Após essa etapa de recuperar o documento e consultar o número de registros, vamos apresentar o conteúdo que ```dados``` recebeu pelo comando ```scala> dados.collect()```.

[Imagem 5]

📢  ETAPA 5: DATAFRAME PARA SQL 

Agora vamos criar uma ```val dfGames``` para trabalhar como SQL. Para realizar ess processo é necessário aplicar o código abaixo e depois consultar com o comando ```scala> dfGames.printSchema```

```bash
scala>  val dfGames = sc.textFile("hdfs://localhost:54310/user/igti/DADOS_GAME/part-m-00000")
```
Vale lembrar que o nosso arquivo não tem cabeçalho, então o Spark tem a capaciade de criar um cabeçalho básico, seguindo a sequencia: 
- [x] _c0 - id
- [x] _c1 - nome_jogo
- [x] _c2 - plataforma
- [x] _c3 - anoLancamento
- [x] _c4 - genero
- [x] _c5 - fabricante
- [x] _c6 - na_sales (Vendas na América)
- [x] _c7 - eu_sales (Vendas na Europa)
- [x] _c8 - jp_sales (Vendas no Japão)
- [x] _c9 - other_sales
- [x] _c10 - global_sales

[Imagem 6]

📢  ETAPA 6: CONSULTAS SQL 

Agora vamos executar algumas sentenças SQL. Mas antes disso, vamos criar uma visão temporária para que possamos manipular os dados do df com SQL. 

```bash
scala>  dfGames.createOrReplaceTempView("DADOS_GAME")
```
Agora é possível consultar os dados pela visão temporária que foi carregada em 'DADOS_GAME' de forma estruturada.

```bash
scala>  spark.sql("SELECT * FROM DADOS_GAME").show(231)
```
[Imagem 7]

Agora vamos aplicar uma consulta para saber o TOTAL GLOBAL por ANO apenas se for maior do que 10 milhões!

```bash
scala>  spark.sql("SELECT * FROM (SELECT _c3, sum(_c10) as total_global from DADOS_GAME group by _c3 order by _c3) as somador where somador.total_global > 10").show(231)
```
[Imagem 8]

Agora vamos propor uma situação de negócios! Quanto cada categoria rendeu em milhões? Para isso vamos aplicar a consulta:

```bash
scala>  spark.sql("SELECT _c4, sum(_c10) from DADOS_GAME group by _c4 order by _c4").show(231)
```
[Imagem 9]

E por último, vamos aplicar outra situação de negócio! Quantos cada fabricante rendeu em milhões entre o ano de 2000 e 2009? Vamos lá, para isso vamos aplicar a consulta:
```bash
scala>  spark.sql("SELECT _c5 , sum(_c10) from DADOS_GAME where _c3 >= 2000 and _c3 <= 2009 group by _c5").show(231)
```
