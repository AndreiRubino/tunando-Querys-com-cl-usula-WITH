# Tunando Querys com cláusula WITH

Este artigo tem como objetivo esclarecer os conceitos e citar exemplos práticos
da Cláusula WITH conhecida como CTE(Commom table expression). As formas abaixo são uteis no dia a dia de cada desenvolvedor/dba.

A CTE é muito utilizada a fim de tunar queries, basicamente ela é uma instrução Select que cria uma espécie de tabela temporária(um result set) com seu resultado. Então em cima desse resultado efetuamos novas instruções SQL e isso facilita muito consultas complicadas. 

Estrutura de uma CTE 
```
WITH cte [ ( column_name [,...n] ) ]     
AS ( 
	CTE_query 
) 
SELECT 
FROM CTE;
```

Digamos que seu gestor pede um relatório para descobrir quem são os funcionários mais antigos de cada departamento atualmente. Na tabela Employees nós temos o Código do departamento e também a data de contratação.

![Image](https://bn1301files.storage.live.com/y4m2OJUBobDEw4_o5drQbuBcoNV14jhOIHs8TxQCoA9P8SEas2GyDjeIdbCyZqkZtwHJcyindmZHxC6G3HcZkUViJ52v7pfFmqFKhYSuVLXv_e5ECC2HAB26rpU0LWaLB_AIYijBGjfLYasIfzXxBEtHixhO5eQuH4e6uHHzJuqyUkZpwdlf5e2WuM_D42CIPDT?width=628&height=300&cropmode=none)


Com a cláusula WITH nós conseguimos em uma única query “rankear” todos os funcionários por departamento, e fazer o select apenas em cima desse resultado visto que a cláusula with cria uma espécie de result set temporário na sua execução. 
```
with cte as (
  select employee_id 
  , department_id
  , hire_date
  , RANK() OVER (PARTITION BY department_id ORDER BY hire_date) rank_contratacao
from hr.employees
)
select 
  cte.employee_id
  , e.first_name
  , cte.department_id
  , cte.hire_date
from cte 
join hr.employees e on e.employee_id = cte.employee_id
where rank_contratacao = 1
order by department_id;
```

Com a query acima nós tivemos o seguinte resultado.

![Image](https://bn1301files.storage.live.com/y4mgnrcQGvvuAOwhzCEuw6hfpxyrQ83ZOrqQts--EJ3p3pflDNpFSJTZ5fHh9d4ryj9k97z6FQKsskIDieXw5Q_81ugrH7fZWAWT7OIk4qEz5YDY-u7OjQP-pWgOTk5UnygDUPLCub-d4O3_v6bTiw0otvK_HwmGQxU4INmgBwoZbEs7gztJFhoE3u249Lfuk5E?width=628&height=292&cropmode=none)

Outra forma de conseguirmos o mesmo resultado seria utilizando uma subquery, no entanto menos performática no caso devido a ser uma subquery correlacionada (mutiple rows) ou seja são executadas linha a linha sobre a query da esquerda(outer query). 
```
select 
  jh1.employee_id 
  , e.first_name
  , jh1.department_id
  , jh1.hire_date
from hr.employees jh1
join hr.employees e on e.employee_id = jh1.employee_id
where not exists (select 
                    1 
                  from hr.employees jh2
                  where 
                    jh2.department_id = jh1.department_id
                  and jh1.hire_date > jh2.hire_date)
order by jh1.department_id;
```

Com a query acima nós tivemos o seguinte resultado.

![Image](https://bn1301files.storage.live.com/y4m0O-H_HTOcz-DQ0MffaLtn9gC4pkcc2bq-oiBZOUlWogy4R6GI3HvkltMsae9OWYM7CyT1g79NSiQVwC7D0R2nUKbfOG_DBHRNq4iofoaVLUHxIBgEW5rz3Jzs3QvVa_opjy3XO_q0G0mbOfnfhfBo7Nq5m7ZREvRnlTX5_04ZB3RIQXec0XFrqPK8AqmtnQg?width=628&height=292&cropmode=none)

Concluímos que com a utilização da clásula WITH temos diversas vantagens como deixar a query mais legível, aumento de performance visto que a CTE reduz o uso de recursos do BD, além de facilitar muito a criação de querys recursivas e querys com window function(row_number(), rank(), dense_rank()).
