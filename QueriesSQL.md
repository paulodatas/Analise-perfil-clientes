``` SQL 
-- (Query 1) Gênero dos leads
-- Colunas: gênero, leads(#)

select
	case
		when ibge.gender = 'male' then 'homens'
		when ibge.gender = 'female' then 'mulheres'
		end as "gênero",
	count(*) as "leads (#)"

from sales.customers as cus
left join temp_tables.ibge_genders as ibge
	on lower(cus.first_name) = lower(ibge.first_name)
group by ibge.gender
```
![query1](https://user-images.githubusercontent.com/124627259/223174522-587dba1d-5d52-4d44-9d8b-a12943a3fb6c.PNG)

``` SQL
-- (Query 2) Status profissional dos leads
-- Colunas: status profissional, leads (%)

select
	case
		when professional_status = 'freelancer' then 'freelancer'
		when professional_status = 'retired' then 'aposentado(a)'
		when professional_status = 'clt' then 'clt'
		when professional_status = 'self_employed' then 'autônomo(a)'
		when professional_status = 'other' then 'outro'
		when professional_status = 'businessman' then 'empresário(a)'
		when professional_status = 'civil_servant' then 'funcionário(a) público(a)'
		when professional_status = 'student' then 'estudante'
		end as "status professional",
	(count(visit_id)::float)/(select count(*) from sales.customers) as "leads (%)"
from sales.customers as cus
left join sales.funnel as fun
	on cus.customer_id = fun.customer_id
group by "status professional"
order by "leads (%)"
``` 
![query2](https://user-images.githubusercontent.com/124627259/223174583-b2cdd8c5-afef-40be-a7eb-77ad53fd7943.PNG)

``` SQL
	
-- (Query 3) Faixa etária dos leads
-- Colunas: faixa etária, leads (%)

SELECT 
    CASE
        WHEN date_part('year', age(current_date, birth_date)) < 20 THEN '0-20'
        WHEN date_part('year', age(current_date, birth_date)) < 40 THEN '20-40'
        WHEN date_part('year', age(current_date, birth_date)) < 60 THEN '40-60'
        WHEN date_part('year', age(current_date, birth_date)) < 80 THEN '60-80'
        ELSE '80+'
    END AS "faixa etária",
    COUNT(*)::float/(SELECT COUNT(*) FROM sales.customers) AS "leads (%)"
FROM sales.customers
GROUP BY "faixa etária"
ORDER BY "faixa etária" desc
```
![query3](https://user-images.githubusercontent.com/124627259/223174661-95029d0e-b916-4cc5-8102-49a0765b3836.PNG)

``` SQL
-- (Query 4) Faixa salarial dos leads
-- Colunas: faixa salarial, leads (%), ordem

SELECT 
    CASE
        WHEN income < 5000 THEN '0-5000'
        WHEN income < 10000 THEN '5000-10000'
        WHEN income < 15000 THEN '10000-15000'
        WHEN income < 20000 THEN '15000-20000'
        ELSE '20000+'
    END AS "faixa salarial",
    COUNT(*)::float/(SELECT COUNT(*) FROM sales.customers) AS "leads (%)",
	CASE
        WHEN income < 5000 THEN 1
        WHEN income < 10000 THEN 2
        WHEN income < 15000 THEN 3
        WHEN income < 20000 THEN 4
        ELSE 5
    END AS "ordem"
FROM sales.customers
GROUP BY "faixa salarial", "ordem"
ORDER BY "ordem" desc
```
![query4](https://user-images.githubusercontent.com/124627259/223174702-93931895-4c72-4035-bade-8da2792655d4.PNG)

``` SQL
	
	-- (Query 5) Classificação dos veículos visitados
-- Colunas: classificação do veículo, veículos visitados (#)
-- Regra de negócio: Veículos novos tem até 2 anos e seminovos acima de 2 anos

with
	classificacao_veiculos as (
		
		select
			fun.visit_page_date,
			pro.model_year,
			extract('year' from visit_page_date) - pro.model_year::int as idade_veiculo,
		case
			when (extract('year' from visit_page_date) - pro.model_year::int)<=2 then 'novo'
			else 'seminovo'
			end as "classificação do veiculo" 
		from sales.funnel as fun
		left join sales.products as pro
			on fun.product_id = pro.product_id
	)
select
	"classificação do veiculo",
	count(*) as "veículos visitados (#)"
from classificacao_veiculos
group by "classificação do veiculo"
```
![query5](https://user-images.githubusercontent.com/124627259/223174765-725ec113-f6f2-4d9c-9376-c27e10c6be4e.PNG)

``` SQL
	
-- (Query 6) Idade dos veículos visitados
-- Colunas: Idade do veículo, veículos visitados (%), ordem

with
	faixa_de_idade_dos_veiculos as (
		
		select
			fun.visit_page_date,
			pro.model_year,
			extract('year' from visit_page_date) - pro.model_year::int as idade_veiculo,
		case
			when (extract('year' from visit_page_date) - pro.model_year::int)<=2 then 'até 2 anos'
			when (extract('year' from visit_page_date) - pro.model_year::int)<=4 then 'de 2 à 4 anos'
			when (extract('year' from visit_page_date) - pro.model_year::int)<=6 then 'de 4 à 6 anos'
			when (extract('year' from visit_page_date) - pro.model_year::int)<=8 then 'de 6 à 8 anos'
			when (extract('year' from visit_page_date) - pro.model_year::int)<=10 then 'de 8 à 10 anos'
			else 'acima de 10 anos' 
			end as "idade do veiculo",
		case
			when (extract('year' from visit_page_date) - pro.model_year::int)<=2 then 1
			when (extract('year' from visit_page_date) - pro.model_year::int)<=4 then 2
			when (extract('year' from visit_page_date) - pro.model_year::int)<=6 then 3
			when (extract('year' from visit_page_date) - pro.model_year::int)<=8 then 4
			when (extract('year' from visit_page_date) - pro.model_year::int)<=10 then 5
			else 6 end as "ordem" 
		
		from sales.funnel as fun
		left join sales.products as pro
			on fun.product_id = pro.product_id
	)
select
	"idade do veiculo",
	count(*)::float/(select count(*) from sales.funnel) as "veículos visitados (%)"
from faixa_de_idade_dos_veiculos
group by "idade do veiculo", ordem
order by ordem
```
![query6](https://user-images.githubusercontent.com/124627259/223174814-2df4f4e9-648a-4719-83ff-6b68b9b00852.PNG)

``` SQL

-- (Query 7) Veículos mais visitados por marca
-- Colunas: brand, model, visitas (#)



select
	pro.brand,
	pro.model,
	count(*) as "visitas (#)"
	
from sales.funnel as fun
left join sales.products as pro
	on fun.product_id = pro.product_id
group by pro.brand, pro.model
order by pro.brand, pro.model, "visitas (#)"
```
![query7](https://user-images.githubusercontent.com/124627259/223174860-db80bd98-a561-4a2e-aeaf-278b9280d39c.PNG)
