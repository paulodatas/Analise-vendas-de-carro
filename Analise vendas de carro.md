
 * [SQL](https://www.microsoft.com/pt-br/sql-server/sql-server-downloads)
 
``` SQL 
-- (Query 1) Receita, leads, conversão e ticket médio mês a mês
-- Colunas: mês, leads (#), vendas (#), receita (k, R$), conversão (%), ticket médio (k, R$)

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
 
