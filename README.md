# Análise de dados - Brazilian E-commerce Olist Store

# Introdução

Este é um conjunto de dados públicos de comércio eletrônico brasileiro de pedidos feitos na Olist Store. O conjunto de dados contém informações de 100 mil pedidos de 2016 a 2018 feitos em vários marketplaces no Brasil. Seus recursos permitem visualizar um pedido de várias dimensões: desde o status do pedido, preço, pagamento e desempenho do frete até a localização do cliente, atributos do produto e, finalmente, avaliações escritas pelos clientes. Também lançamos um conjunto de dados de geolocalização que relaciona os códigos postais brasileiros às coordenadas latitude/longitude.

Estes são dados comerciais reais, foram anonimizados e as referências às empresas e parceiros no texto da revisão foram substituídas pelos nomes das grandes casas de Game of Thrones.


# Contexto

Este conjunto de dados foi generosamente fornecido pela Olist, a maior loja de departamentos do mercado brasileiro, e coletado na plataforma Kaggle.

Depois que um cliente compra o produto da Olist Store, um vendedor é notificado para atender a esse pedido. Assim que o cliente recebe o produto, ou a data prevista de entrega está prevista, o cliente recebe um inquérito de satisfação por e-mail onde pode dar nota da experiência de compra e deixar alguns comentários.

Atenção

1 - Um pedido pode ter vários itens.
2 - Cada item pode ser atendido por um vendedor distinto.
3 - Todo o texto identificando lojas e parceiros foi substituído pelos nomes das grandes casas de Game of Thrones.
4 - Os dados não incluem grandes centros do país como São Paulo e Rio de Janeiro, assim como a região Sul do Brasil.

# Modelagem de lógica

Os dados são divididos em vários conjuntos de dados para melhor compreensão e organização. Consulte o esquema de dados a seguir ao trabalhar com ele:

![Modelagem](https://user-images.githubusercontent.com/78854666/211364199-973141a4-60fe-4743-b5de-e4de7efc61c3.png)

<div align="center">
<img src="https://user-images.githubusercontent.com/78854666/211364199-973141a4-60fe-4743-b5de-e4de7efc61c3.png" width="0px" />
</div>


# Construindo tabelas no Google Bigquery

Boa parte do ETL foi realizado via linguagem SQL no Google Bigquery. 
Inicialmente, importei no Google Bigquery as tabelas em disponibilizadas no site Kaggle. Realizei consultas para avaliar quais tabelas e colunas seriam necessárias para responder perguntas relacionadas à performance de vendas por cliente em determinado período. Sendo assim, usarei apenas as tabelas ‘orders’, 'orders items' e 'order customer'.

Posteriormente, criei um ambiente de produção, criando tabelas idênticas às que selecionei na etapa anterior. Sendo assim, realizei as seguintes consultas:

CREATE OR REPLACE TABLE Sales.ACT_CUSTOMER AS
(
SELECT *
FROM Sales.customer
);

CREATE OR REPLACE TABLE Sales.ACT_ORDER AS
(
SELECT *
FROM Sales.order
);

CREATE OR REPLACE TABLE Sales.ACT_ORDER_ITEMS AS
(
SELECT *
FROM Sales.order.items
)
 
Após criar as tabelas do ambiente de produção, selecionei apenas as colunas(dimensões e métricas) que julguei necessárias para a realização das análises. Foi necessária a realização do  INNER JOIN em duas etapas, já que as três tabelas não tinham dimensões em comum simultaneamente . A primeira etapa foi com o objetivo de juntar as tabelas 'ACT ORDERS' e ‘ACT CONSUMER'. Em seguida, criei uma visualização com a JOIN anterior, chamando-a de ‘VW_PEDIDOS_POR_CLIENTE’.

Na segunda etapa, realizei um INNER JOIN  com a visualização criada anteriormente e a tabela ‘ACT ORDER ITEMS’ que havia faltado, concluindo assim a etapa de tratamento da base de dados.

Para isso realizei as seguintes consultas:

SELECT O. *, C.customer_city, C.customer_state
FROM Sales.ACT_ORDERS AS O
INNER JOIN Sales.ACT_CUSTOMER AS C
ON O.customer_id = C.customer_id

SELECT PC. *, OI.price, OI.freight_value
FROM Sales.VW_PEDIDOS_POR_CLIENTE AS PC
INNER JOIN Sales.ACT_ORDER_ITEMS AS OI
ON PC.order_id = OI.order_id

<h3 align="left">Languages and Tools:</h3>
<p align="left"> <a href="https://www.figma.com/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/figma/figma-icon.svg" alt="figma" width="40" height="40"/> </a> <a href="https://cloud.google.com" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/google_cloud/google_cloud-icon.svg" alt="gcp" width="40" height="40"/> </a> <a href="https://www.microsoft.com/en-us/sql-server" target="_blank" rel="noreferrer"> <img src="https://www.svgrepo.com/show/303229/microsoft-sql-server-logo.svg" alt="mssql" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> </p>


# Dataviz em Tableau

Para a visualização de dados, utilizei o Tableau, ferramenta extremamente intuitiva que nos permite criar dashboards dinâmicos e que possui a facilidade de acessá-lo de forma online, sem a necessidade de download do arquivo. 

![Ollist ptf](https://user-images.githubusercontent.com/78854666/211379478-f50c8f06-ddb6-469e-82a1-28f0b38d6936.png)

<div align="center">
<img src="https://user-images.githubusercontent.com/78854666/211379478-f50c8f06-ddb6-469e-82a1-28f0b38d6936.png" width="0px" />
</div>

A ideia com a construção do dashboard foi demonstrar de forma simples e objetiva os principais resultados da Olist relacionados às vendas. A partir dessa visão, foram respondidas algumas perguntas importantes, e além disso, aprendizados importantes com relação a alguns comportamentos de compra por parte dos usuários. 

Acesse o dashboard completo em https://public.tableau.com/app/profile/lincoln.mac.do/viz/Ptf-EcommerceOlist/Dashboard


# Análise

Em 2018, houve um aumento de vendas superior a 12% com relação a 2017. E na contramão, o número de compras canceladas caiu 8% nesse período. Podendo indicar eficiência com relação aos serviços oferecidos, como entrega, atendimento, assim como a qualidade dos produtos disponibilizados na plataforma. Esse crescimento percentual é igual aos 12% em que o setor de e-commerce evoluiu no mesmo período, segundo o Ebit.

Essa evolução foi influenciada pelos resultados de Estados como Minas Gerais, Bahia, Distrito Federal e Goiás, que estão entre os Estados com mais densidade populacional e renda per capita (com exceção da Bahia) do Brasil.

Dos Estados com maior número de vendas, a Bahia foi o que teve o maior ticket médio (R$ 4.437,35), 17% maior que Minas Gerais. O que chama atenção é justamente o fato da Bahia estar entre os estados com menor renda per capita do Brasil, segundo  o IBGE. Alguns fatores podem justificar esse comportamento, como o aumento do consumo.

A tendência é que o mercado digital evolua mais a cada ano, justificado pelo comportamento social em que as compras online têm se tornado um hábito mais comum. Sendo assim, como recomendação, deve-se aumentar 20% o share de mídia digital no próximo ano, continuando focando nos grandes centros, e também nos Estados que demonstraram potencial de compra no ano de 2018. 







