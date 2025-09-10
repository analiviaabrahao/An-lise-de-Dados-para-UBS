## 🏥 Desenvolvimento de um Sistema Automatizado para Gestão e Acompanhamento de Filas em Unidades Básicas de Saúde
##

Este projeto implementa um sistema de gerenciamento de fila de pacientes para uma UBS (Unidade Básica de Saúde), levando em conta:

Grau de Prioridade do paciente

Tipo de exame solicitado

Data de entrada na fila

Com base nessas regras, o sistema calcula e atualiza automaticamente a posição do paciente na fila.
Além de mostrar gráficos com informações úteis para a tomada de decisão do gestor

##

## 🚀 Tecnologias Utilizadas

Next.js (API Routes e frontend)
Shadcn ui (frontend)

PostgreSQL (banco de dados relacional)

Node.js (ambiente de execução)

SQL (para cálculo e consulta da posição na fila)

## Como funciona o cálculo da fila ⚙️
A posição é calculada usando window function do PostgreSQL:

SELECT 
  f.id,
  p.nome,
  e.tipo_exame,
  p.prioridade,
  p.data_entrada,
  ROW_NUMBER() OVER (
    PARTITION BY e.tipo_exame, p.prioridade
    ORDER BY p.data_entrada ASC
  ) AS posicao_fila
FROM fila f
JOIN paciente p ON f.paciente_id = p.id
JOIN exame e ON e.paciente_id = p.id;


ORDER BY → define a ordem pela data de entrada

ROW_NUMBER() → gera a posição do paciente na fila


## Para Calcular e Atualizar a posição no banco ⚙️

Uso o ROW_NUMBER() do SQL para calcular a posição dentro de cada grupo (prioridade + tipo_exame) e fazer o UPDATE:

await pool.query(`
  WITH fila_ordenada AS (
    SELECT
      f.id,
      ROW_NUMBER() OVER (
        PARTITION BY f.prioridade, e.tipo_exame
        ORDER BY f.data_entrada ASC
      ) AS posicao_fila
    FROM fila f
    LEFT JOIN exame e ON e.paciente_id = f.paciente_id
  )
  UPDATE fila f
  SET posicao_fila = fo.posicao_fila
  FROM fila_ordenada fo
  WHERE f.id = fo.id;
`);
<div>

        <img src="https://drive.google.com/file/d/1HD3t3iANnoj06HQuEKx1p8xWvuIZhCv8/view?usp=sharing" alt="Texto Alternativo" width="100">
</div>






















  f.data_entrada ASC, -- Ordem de chegada
  e.tipo_exame ASC;   -- Tipo de exame
