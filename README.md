# Analise de Dados do E-commerce Olist no SQL

## Origem dos Dados
### A base de dados utilizada para reproduzir os conhecimentos é da [Olist](https://www.kaggle.com/olistbr/brazilian-ecommerce).

Esse é um conjunto de dados público do comércio eletrônico brasileiro e possui cerca de 100 mil registros referentes ao ano de 2016 a 2018, algumas das informações disponíveis são:
- **status do pedido**
- **preço**
- **atributos do produto**
- **avaliações feitas pelo cliente**
  

## **Schema**

![schema](https://github.com/user-attachments/assets/fec0c7fb-4ebe-42bc-b6cb-324794eed9d2)

**Ferramentas**

-   No projeto utilizei a IDE DBeaver para praticar as consultas.
-   Disponivel em: (DBeaver)  https://dbeaver.com/

# O problema de Negócio
O CEO está focado em obter uma análise detalhada e estratégica sobre diversos aspectos operacionais da empresa, que tem a necessidade de entender melhor o perfil dos clientes com base em sua localização ( Santa Catarina ), preferências de pagamento e comportamento de compra, além de garantir uma gestão eficiente da logística e dos produtos de maior valor. Ao solicitar:
- **Informações sobre a localização dos clientes (cidade e estado), o CEO busca entender a distribuição geográfica da base de consumidores e identificar oportunidades de segmentação regional.**
- **Dados dos clientes que vivem em estados ou cidades específicas, como Santa Catarina e Florianópolis, ajudam a identificar regiões prioritárias e possíveis melhorias em estratégias locais.**
- **Detalhes sobre pagamentos parcelados permitirão ao CEO avaliar riscos financeiros, como inadimplência, além de ajustar a estratégia de crédito e otimizar o fluxo de caixa da empresa.**
  
Esses dados ajudarão o CEO a tomar decisões mais informadas, tanto no nível de operação quanto em questões estratégicas, buscando aumentar a eficiência, a lucratividade e a experiência dos clientes.


### Perguntas do CEO
#### visão clientes
#### Quais são as 5 cidades com mais clientes, no estado de Santa Catarina?

```sql
SELECT
	count( c.customer_id ) as qtd_clientes, 
	c.customer_city  as cidade,
	c.customer_state as UF
FROM customer c  
WHERE c.customer_state = 'SC'
GROUP BY c.customer_city 
ORDER BY qtd_clientes DESC 
LIMIT 5
```
### Resultado da query
|qtd_clientes | cidade | UF|
|:------------:|:-------|--:|
|570	|florianopolis |	SC|
| 264|	joinville|	SC|
|188|	blumenau	|SC|
| 169	|sao jose|	SC|
|120|	itajai	|SC|

### Insight
- Joinville tem o dobro do PIB de Florianópolis e população é maior também, então acredito que espandir a base de clientes com campanhas de marketing aumentará o faturamento da Olist.

#### Visão produtos
#### Quais as categorias de produtos mais vendidos por cidade no estado de Santa Catarina?

```sql
SELECT 
	count ( o.order_id ) as qtd_pedidos,
	p.product_category_name as categorias,
	c.customer_city as cidades, c.customer_state as UF
FROM customer c LEFT JOIN orders o ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi   ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	     ON p.product_id = oi.product_id 
WHERE categorias NOTNULL and c.customer_state = 'SC'
GROUP BY categorias 
ORDER BY qtd_pedidos DESC 
LIMIT 5
```
### Resultado da query
| qtd_pedidos | categorias| cidades| UF|
|:-----------:|:----------|:--------|--:|
|363|	esporte_lazer	|jaragua do sul	|SC|
|354|	moveis_decoracao|	blumenau	|SC|
|330|	cama_mesa_banho|	concordia	|SC|
|320	|informatica_acessorios|	florianopolis|	SC|
|309|	beleza_saude|	navegantes	|SC|

### Insight
- As 5 categorias quem mais vendem no e-commerce estão associadas ao bem-estar e estilo de vida dos clientes entre outros, então a criação de combos como produtos como esportes e saúde ( tenis de corridas e suplementação ) é uma ótima forma de combinar as categorias que mais vendem para aumentar o faturamento.

#### Qual o valor médio das 5 primeiras categorias que mais vendem?

```sql
SELECT 
	round(avg( oi.price )) as avg_price,
	p.product_category_name as categoria
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
WHERE p.product_category_name  NOTNULL and c.customer_state = 'SC'
GROUP BY categoria
ORDER BY avg_price DESC
LIMIT 5
```
### Resultado da query
|avg_price| categorias|
|:-------:|-----------|
|1212.0|	pcs|
|1189.0|	portateis_casa_forno_e_cafe|
|470.0|	eletrodomesticos_2|
|357.0|	agro_industria_e_comercio|
|307.0|	eletroportateis|

### Insight
- Eletronicos e eletrodomésticos estão entre são as duas categorias mais rentaveis do e-commerce, isso mostra que nosso cliente preza pela conveniência das máquinas de café e tecnologia, então priorizar produtos que melhorem a vida no 'homeoffice' é uma ótima idéia.

#### Visão vendedores
#### Quais as 5 cidades que têm mais vendedores?
```sql
SELECT 
	count( oi.seller_id ) as qtd_vendedores,
	c.customer_city  as cidades
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
WHERE p.product_category_name  NOTNULL and c.customer_state = 'SC'
GROUP BY cidades
ORDER BY qtd_vendedores DESC
LIMIT 5
```
### Resultado da query
|qtd_vendedores| cidades|
|:------------:|--------|
|668|	florianopolis|
|292|	joinville|
|223|	blumenau|
|201	|sao jose|
|129|	chapeco|

### Insight
- A cidade onde tem o menor número de vendedores é a que tem o maior poder de compra, então aumentar o número de vendedores e campanha de marketing para essa cidade é uma ótima opção pois Itajaí é a cidade mais rica do estado de Santa Catarina, com um PIB de R$47 bilhões, então o foco incial é aumentar a base de consumidores. Na nossa base de dados há mais vendedores em Chapecó que tem a mesma quantidade de habitantes com um PIB de R$13 bilhões.
  
#### Visão Pedidos
#### Qual a forma de pagamento mais usada?
```sql
SELECT 
	count ( op.payment_type )  as qtd_pagamentos,
	op.payment_type as tipo_pagamento
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
WHERE p.product_category_name  NOTNULL and c.customer_state = 'SC'
GROUP BY tipo_pagamento
ORDER BY qtd_pagamentos DESC
```
|qtd_pagamentos| tipo_pagamento|
|:------------:|:--------------|
|3042	|credit_card|
|987	|boleto|
|160	|voucher|
|48|	debit_card|

### Insight 
- Baixa adesão a pagamentos no cartão de débito, precisamos avaliar se os nossos clientes têm alguma dificuldade em pagar com cartão de crédito no nosso site, se não está tão visível na hora da compra.

#### Qual a quantidade de boletos cancelados? 

```sql
SELECT 
	count( o.order_status ) as status,
	o.order_status 
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
WHERE p.product_category_name  NOTNULL and c.customer_state = 'SC' and o.order_status = 'canceled'
GROUP BY o.order_status 
ORDER BY status DESC
```
### Comentário 
- Em todo estado há apenas 14 casos de cancelamentos.

### Qual o motivo dos cancelamentos
```sql
SELECT
	o.order_status as status,
	o.order_purchase_timestamp as data_compra,
	o.order_approved_at as data_aprovação,
	o.order_delivered_carrier_date as entregue_transportadora ,
	o.order_estimated_delivery_date as entrega_estimada,
	o.order_delivered_customer_date as data_entrega  
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
WHERE p.product_category_name  NOTNULL 
	and c.customer_state = 'SC' 
	and o.order_status = 'canceled'
```
### Comentário
- Percebi que os pedidos cancelados eles têm data de compra, aprovação e de entrega estimada, mas não têm data de entrega a transportadora.

| status | data_compra        |   data_aprovacao| entregue_transportadora | entrega_estimada |data_entrega |
|:-------|--------------------|-----------------|:----------------------:|-------------------|:------------------:|
|canceled|	2017-03-11 19:51:36	|2017-03-11 19:51:36|		  [NULL]               |   2017-03-30 00:00:00|	  [NULL]|
|canceled|	2017-09-13 20:38:00	|2017-09-13 20:50:12		|  [NULL]                |2017-10-11 00:00:00	|     [NULL]|
|canceled|	2017-07-05 21:47:36	|2017-07-05 22:03:30|		   [NULL]            |    2017-07-27 00:00:00	   |  [NULL]|
|canceled	|2018-02-03 13:37:35|	2018-02-03 13:50:26	|2018-02-05 19:59:59	    | 2018-03-01 00:00:00	     |[NULL]|
|canceled|	2018-02-25 21:11:16	|2018-02-25 21:25:23|		   [NULL]           |       2018-03-19 00:00:00	  | [NULL]|
|canceled	|2017-03-24 00:24:17	|2017-03-24 01:23:01		 |   [NULL]               | 2017-04-26 00:00:00	    | [NULL]|
|canceled|	2017-05-17 15:46:06|	2017-05-17 16:02:33	|	    [NULL]         |           2017-06-08 00:00:00	 |  [NULL]|
|canceled|	2017-05-17 15:46:06	|2017-05-17 16:02:33		|    [NULL]       |             2017-06-08 00:00:00	  | [NULL]|
|canceled|	2017-04-10 00:45:56|	2017-04-10 01:03:29		 |   [NULL]          |          2017-05-04 00:00:00	   |[NULL]|
|canceled	|2016-10-09 15:39:56	|2016-10-10 10:40:49|	2016-10-14 10:40:50	  |    2016-12-08 00:00:00	 |2016-11-09 14:53:50|
|canceled	|2017-07-24 13:22:29|	2017-07-27 10:10:15		|     [NULL]   |                2017-08-04 00:00:00	  | [NULL]|
|canceled	|2017-07-24 13:22:29	|2017-07-27 10:10:15		|     [NULL]   |                2017-08-04 00:00:00	 |  [NULL]|
|canceled|	2018-05-05 13:36:10|	2018-05-05 13:50:25	|	     [NULL]    |               2018-05-17 00:00:00	 |  [NULL]|
|canceled	|2017-08-25 19:10:57|	2017-08-29 04:26:12		|   [NULL]   |                2017-09-20 00:00:00	  | [NULL]|


```sql
SELECT 
	or2.review_score as avaliacao,
	o.order_estimated_delivery_date as entrega_estimada,
	or2.review_creation_date as data_comentario,
	or2.review_comment_message as comentario,
	or2.review_answer_timestamp as reposta_empresa
FROM customer c LEFT JOIN orders o        ON o.customer_id = c.customer_id 
				LEFT JOIN order_items oi  ON oi.order_id  = o.order_id 
				LEFT JOIN products p 	  ON p.product_id = oi.product_id 
				LEFT JOIN order_payments op ON o.order_id = op.order_id 
				LEFT JOIN order_reviews or2 ON or2.order_id = o.order_id 
WHERE p.product_category_name  NOTNULL 
	and c.customer_state = 'SC' 
	and o.order_status = 'canceled'
```
### Observando a avaliação e comentários dos clientes que cancelaram as compras
|avaliação| entrega_estimada | data_comentario | comentario_clientes | resposta_empresa|
|:--:|--------------------|------------------|--------------|-----------------|  
|1	  |  2017-03-30 00:00:00	|2017-04-01 00:00:00|		[NULL]            |2017-04-01 10:24:03|
|4	|2017-10-11 00:00:00	|2017-10-13 00:00:00|		     [NULL]           |2017-10-13 09:49:26|
|1	|2017-07-27 00:00:00|	2017-07-29 00:00:00|	Nem a nota fiscal foi emitida||	2017-07-29 05:07:46|
|1|	2018-03-01 00:00:00	|2018-03-03 00:00:00	|	      [NULL]            |2018-03-03 04:01:02|
|1	|2018-03-19 00:00:00	|2018-03-21 00:00:00	|	      [NULL]            |2018-03-21 09:20:40|
|1|	2017-04-26 00:00:00	|2017-04-28 00:00:00|		        [NULL]          |2017-04-29 13:33:27|
|1	|2017-06-08 00:00:00	|2017-06-10 00:00:00	|Não entregou o produto e não entrou em contato.	|2017-06-10 23:31:07|
|1	|2017-06-08 00:00:00	|2017-06-10 00:00:00	|Não entregou o produto e não entrou em contato.	|2017-06-10 23:31:07|
|1	|2017-05-04 00:00:00	|2017-05-06 00:00:00	|O produto não veio	|2017-05-06 05:05:21|
|1	|2016-12-08 00:00:00	|2016-11-10 00:00:00	|consta que meu pedido foi entregue mas nao chegou ha possibilidade de informar assinatura de quem recebeu no local???	|2016-11-16 20:44:19|
|1	|2017-08-04 00:00:00|	2017-08-11 00:00:00	|Quero que resolvam o meu problema!!!|	2017-08-14 13:32:51|
|1	|2017-08-04 00:00:00	|2017-08-11 00:00:00	|Quero que resolvam o meu problema!!!|	2017-08-14 13:32:51|
|5	|2018-05-17 00:00:00	|2018-05-19 00:00:00	|EU NÃO RECEBI O PRODUTO POIS O VENDEDOR NÃO PODE FATURAR EM NOME DA EMPRESA COM O CNPJ. SOLICITEI CANCELAMENTO DA COMPRA.	|2018-05-21 12:32:09|
|1|	2017-09-20 00:00:00|	2017-09-24 00:00:00	|Insatisfação total. Paguei o produto prontamente, depois de quase um mês recebi a mensagem que o produto estava indisponível.|	2017-09-24 14:48:56|

### Possível problema
- A maioria das reclamações é de não recebemento do produto, mas consta que foi entregue ao cliente, as datas das reclamações são após a suposta data de entrega.
- Provavelmente a empresa tem algum problema no sistema de estoque pois a compra é aprovada e é estimada uma data de entrega, mas o produto não é entregue 
por não ter disponibilidade no estoque, e isso não está sendo repassado ao cliente no site na hora da compra.
