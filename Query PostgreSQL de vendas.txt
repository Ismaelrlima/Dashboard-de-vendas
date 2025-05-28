-- (Query 1) Receita, leads, conversão e ticket médio mês a mês
-- Colunas: mês, leads (#), vendas (#), receita (k, R$), conversão (%), ticket médio (k, R$)
select * from sales.products
select * from sales.funnel

with visitantes as(
select 
	date_trunc('month', visit_page_date)::date as mes_a_mes,
	count (*) as visitas
from sales.funnel
group by mes_a_mes
order by mes_a_mes
),

	vendas as (
		select 
			date_trunc('month', paid_date)::date as mes_a_mes,
			count(paid_date) as vendas_mes,
			sum(pro.price * (1+fun.discount))/1000 as receita
		from sales.funnel as fun 
		left join sales.products as pro
			on fun.product_id = pro.product_id
		where paid_date is not null
		group by mes_a_mes
		order by mes_a_mes
		)

select 
	vis.mes_a_mes as "mês",
	vis.visitas as "leads (#)",
	ven.vendas_mes as "vendas (#)",
	ven.receita as "receita (k, R$)",
	ROUND(((ven.vendas_mes::float / vis.visitas::float) * 100)::numeric, 2) AS "conversão (%)",
    ROUND((ven.receita / ven.vendas_mes)::numeric, 2) AS "ticket médio (k, R$)"
from visitantes as vis
left join vendas as ven
	on vis.mes_a_mes = ven.mes_a_mes
group by "mês","leads (#)","vendas (#)","receita (k, R$)"
	

-- (Query 2) Estados que mais venderam
-- Colunas: país, estado, vendas (#)

select * from sales.customers
select * from sales.funnel
select * from temp_tables.regions

select
	'Brasil' as pais,
	cus.state as estado,
	count (paid_date) as vendas
from sales.funnel as fun
left join sales.customers as cus
	on fun.customer_id = cus.customer_id
where paid_date is not null
group by pais,estado
order by vendas desc



-- (Query 3) Marcas que mais venderam no mês
-- Colunas: marca, vendas (#)
select * from sales.products

select 
	pro.brand as marcas,
	count(fun.paid_date) as contagem
from sales.funnel as fun
left join sales.products as pro
	on fun.product_id = pro.product_id
where paid_date is not null
group by pro.brand
order by contagem desc
limit 10




-- (Query 4) Lojas que mais venderam
-- Colunas: loja, vendas (#)
select * from sales.stores

select
	sto.store_name as  lojas,
	count(paid_date) as vendas
from sales.funnel as fun
left join sales.stores as sto
	on fun.store_id = sto.store_id
group by lojas
order by vendas desc
limit 10



-- (Query 5) Dias da semana com maior número de visitas ao site
-- Colunas: dia_semana, dia da semana, visitas (#)

select 
	extract('dow' from visit_page_date) as dia_semana,
		case 
			when extract('dow' from visit_page_date) = 0 then 'Domingo'
			when extract('dow' from visit_page_date) = 1 then 'Segunda'
			when extract('dow' from visit_page_date) = 2 then 'Terça'
			when extract('dow' from visit_page_date) = 3 then 'Quarta'
			when extract('dow' from visit_page_date) = 4 then 'Quinta'
			when extract('dow' from visit_page_date) = 5 then 'Sexta'
			when extract('dow' from visit_page_date) = 6 then 'Sabado'
		else null
		end as "Dia da semana",
		count (visit_page_date) as visitas
from sales.funnel
group by dia_semana,"Dia da semana"
order by visitas desc



