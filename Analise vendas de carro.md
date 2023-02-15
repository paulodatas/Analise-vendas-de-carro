
 * [SQL](https://www.microsoft.com/pt-br/sql-server/sql-server-downloads)
 
``` SQL 


-- (Query 1) Receita, leads, conversão e ticket médio mês a mês
-- Colunas para serem geradas : mês, leads (#), vendas (#), receita (k, R$), conversão (%), ticket médio (k, R$)

-- visitas por mês na pagina 
with
	leads as (
		select date_trunc('month', visit_page_date)::date as visit_page_month,
			count(*) as visit_page_count
		from sales.funnel
		group by visit_page_month
		order by visit_page_month
	), 	

-- receita da empresa por mês 
	payments as (
		select
			date_trunc('month', fun.paid_date)::date as paid_month,
			count(fun.paid_date) as paid_count,
			sum(pro.price * (1 + fun.discount)) as receita
		from sales.funnel as fun
		left join sales.products as pro
			on fun.product_id = pro.product_id
		where fun.paid_date is not null
		group by paid_month
		order by paid_month
	)
-- junção das duas queries anteriores para finalização da conversão dos leads e ticket médio 
select 
	leads.visit_page_month as "mês",
	leads.visit_page_count as "leads (#)",
	payments.paid_count as "vendas (#)",
	(payments.receita/1000) as "receita (K, R$)",
	(payments.paid_count::float/leads.visit_page_count::float) as "conversão (%)",
	(payments.receita/payments.paid_count/1000) as "ticket médio ()"

from leads 
left join payments
	on leads.visit_page_month = paid_month
```
 ![query1](https://user-images.githubusercontent.com/124627259/219187903-55270acf-16dc-481b-b37d-022292a46fb2.PNG)
 
 ----------------------------------------------------------------------------------------------------------------------------

``` SQL
-- (Query 2) Estados que mais venderam
-- Colunas: páis, estado, vendas (#)
	
select 
	'Brazil' as país,
	cust.state as estado,
	count(fun.paid_date) as vendas
	
from sales.funnel as fun
left join sales.customers as cust
	on fun.customer_id = cust.customer_id
where paid_date between '2021-08-01' and '2021-08-31' 
group by estado
order by vendas desc
limit 5

```
![query2](https://user-images.githubusercontent.com/124627259/219196511-93617fcc-bd26-46a6-892b-d063a16c7a07.PNG)

``` SQL

-- (Query 3) 5 marcas que mais venderam no mês
-- colunas: marca, vendas

select 
	prod.brand as marca,
	count(fun.paid_date) as vendas
from sales.funnel as fun
left join sales.products as prod
	on fun.product_id = prod.product_id
where paid_date between '2021-08-01' and '2021-08-31' 
group by marca
order by vendas desc
limit 5
![image](https://user-images.githubusercontent.com/124627259/219198086-bec6a4da-e729-4d8d-8155-530c7b405230.png)


```
![query3](https://user-images.githubusercontent.com/124627259/219199239-da0aa4a2-38e6-4868-923a-93c8aef97138.PNG)

``` SQL

-- (Query 4) Lojas que mais venderam
-- Colunas: loja, vendas (#)

-- (Query 4) 5 lojas que mais venderam no mês
-- colunas: marca, vendas

select 
	stor.store_name as lojas,
	count(fun.paid_date) as vendas
from sales.funnel as fun
left join sales.stores as stor
	on fun.store_id = stor.store_id
where paid_date between '2021-08-01' and '2021-08-31' 
group by lojas
order by vendas desc
limit 5

```
![query4](https://user-images.githubusercontent.com/124627259/219200327-6c5d5a6e-51de-43a3-b158-a1eedf746eef.PNG)

``` SQL 
-- (Query 5) Dias da semana com maior número de visitas ao site
-- Colunas: dia_semana, dia da semana, visitas (#)


select 
	extract('dow' from  visit_page_date) as dia_semana,
	case 
		when extract('dow' from  visit_page_date) = 0 then 'Domingo'
		when extract('dow' from  visit_page_date) = 1 then 'Segunda'
		when extract('dow' from  visit_page_date) = 2 then 'Terça'
		when extract('dow' from  visit_page_date) = 3 then 'Quarta'
		when extract('dow' from  visit_page_date) = 4 then 'Quinta'
		when extract('dow' from  visit_page_date) = 5 then 'Sexta'
		when extract('dow' from  visit_page_date) = 6 then 'Sabado'
		else null end as "dia da semana",
		count(visit_page_date) as "visitas (#)"
from sales.funnel
where visit_page_date between '2021-08-01' and '2021-08-31'
group by dia_semana
order by dia_semana

```
![query5](https://user-images.githubusercontent.com/124627259/219201799-3c5d7393-0093-4de3-a0cc-1f3afd8d06be.PNG)

[Projeto+1+-+Dashboard+de+vendas.xlsx](https://github.com/paulodatas/SQL/files/10748585/Projeto%2B1%2B-%2BDashboard%2Bde%2Bvendas.xlsx)

