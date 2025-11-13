# Oficina: Introdução a análise de dados educacionais

Bem-vindo(a) à página oficial da nossa oficina! Aqui você encontrará todos os materiais de apoio necessários para cada encontro.

**Instrutor(a):** Leidiane Santos

---

### 📅 Cronograma e Materiais
* **Materiais**
    * [Cronograma](cronograma_oficina_dados.pdf)
    * [Base dos Dados](https://basedosdados.org/)    

* **Encontro 1: Fundamentos de dados**
    * [Apresentação de Slides](mod1_1.pdf)
    * [Exemplos de dados educacionais](apoio_mod1.pdf) 
    
* **Encontro 2: Onde Vivem os Dados Educacionais?**
    * [Apresentação de Slides](mod1_2.pdf)
   
* **Encontro 3: Explorando sem código**
   *  [QEdu](https://qedu.org.br/)
   *  [Municípios do Pará](municipios_pa.xlsx)
   *  [Atividade prática - Distorção idade-série Anos Finais-PA](distorcao_idade_serie_pa_2023-AF.xlsx)
   *  [Atividade prática - IDEB - Anos Finais - PA](ideb_territorios-15-2023-AF.xlsx)
   *  **Tarefa: Registrar conta no BigQuery**
   * [Orientações](https://basedosdados.org/docs/access_data_bq)
     

   <img width="1136" height="391" alt="Captura de tela de 2025-10-16 10-44-27" src="https://github.com/user-attachments/assets/38305027-a7e7-455b-9a73-d4c39f9d6958" />

  Você deve cadastrar uma conta no serviço [BigQuery (Google)](https://console.cloud.google.com) 

* **Encontro 4: Grande volumes de dados**
   *  [Atividade prática](scripts)

* **Encontro 5: Primeira consulta**
* * [Aprender SQL- Este site do W3Schools contém um tutorial abrangente para aprender SQL](https://www.w3schools.com/sql/default.asp)
    
* **Encontro 6: Filtrando dados educacionais do Pará**

* **Encontro 7: Questões de pesquisa**
*  [Atividade prática - Scripts completos de exemplo](aula_final.txt)
*  [Passo a passo de como construir a consulta](legenda_query.pdf)

* **Encontro 8: Como construir a visualização**

* **Encontro 9: Apresentação de mini projetos**

* **Códigos SQL Utilizados na Oficina**
    * [Ver todos os scripts na pasta 'Scripts_SQL'](scripts)

---

## 🛠️ Scripts SQL

## Selecionar colunas específicas:

```sql
SELECT
  ano,
  sigla_uf,
  id_escola
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
LIMIT 10;
```

---

## Filtrar apenas escolas do Amapá (AP):

```sql
SELECT
  ano,
  sigla_uf,
  id_escola
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2024
AND sigla_uf = 'AP'
LIMIT 10;
```

**O que acontece se eu substituir por 'MA'?**

---

## Contar as escolas por estado

```sql
SELECT
  sigla_uf,
  COUNT(*) AS total_escolas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
  AND sigla_uf IN ('PA', 'AP', 'MA')
GROUP BY sigla_uf
ORDER BY total_escolas DESC;
```

> **Interpretação:** quantas escolas foram registradas no Censo Escolar 2022 em cada estado selecionado.

---

## Somar o total de matriculas

```sql
SELECT
  sigla_uf,
  SUM(quantidade_matricula_educacao_basica) AS total_matriculas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
  AND sigla_uf IN ('PA', 'AP', 'MA')
GROUP BY sigla_uf;
```

---

## Total matrícula por rede

```sql
SELECT
  sigla_uf,
  rede,
  SUM(quantidade_matricula_educacao_basica) AS matriculas_rede
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
  AND sigla_uf IN ('PA', 'AP', 'MA')
GROUP BY sigla_uf, rede
ORDER BY sigla_uf, rede;
```

---

## Escolas urbanas e rurais

```sql
SELECT
  sigla_uf,
  tipo_localizacao,
  COUNT(id_escola) AS total_escolas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
  AND sigla_uf IN ('PA', 'AP', 'MA')
GROUP BY sigla_uf, tipo_localizacao
ORDER BY sigla_uf, tipo_localizacao;
```

---

## Quantidade de matrícula por município vários anos

```sql
SELECT
  ano,
  id_municipio,
  rede,
  SUM(quantidade_matricula_medio) AS total_matriculas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE sigla_uf = 'PA'
  AND ano BETWEEN 2022 AND 2024
GROUP BY ano, id_municipio, rede
ORDER BY ano, id_municipio, rede;
```

---

## Evolução total sem rede

```sql
SELECT
  ano,
  rede,
  SUM(quantidade_matricula_medio) AS total_matriculas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE sigla_uf = 'PA'
  AND ano BETWEEN 2022 AND 2024
GROUP BY ano, rede
ORDER BY ano, rede;
```

---

## Bônus: Resultado com Brasil UNION ALL

```sql
SELECT
  sigla_uf,
  tipo_localizacao,
  COUNT(id_escola) AS total_escolas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
  AND sigla_uf = 'PA'
GROUP BY sigla_uf, tipo_localizacao

UNION ALL

SELECT
  'BRASIL' AS sigla_uf,
  tipo_localizacao,
  COUNT(id_escola) AS total_escolas
FROM `basedosdados.br_inep_censo_escolar.escola`
WHERE ano = 2022
GROUP BY tipo_localizacao

ORDER BY sigla_uf, tipo_localizacao;
```

---

## Bônus: JOIN para o nome do município 2024

```sql
SELECT
  esc.ano,
  esc.id_municipio,
  mun.nome AS nome_municipio,
  esc.rede,
  SUM(esc.quantidade_matricula_medio) AS total_matriculas
FROM `basedosdados.br_inep_censo_escolar.escola` AS esc
LEFT JOIN `basedosdados.br_bd_diretorios_brasil.municipio` AS mun
  ON esc.id_municipio = mun.id_municipio
WHERE esc.sigla_uf = 'PA'
  AND esc.ano = 2024
GROUP BY esc.ano, esc.id_municipio, mun.nome, esc.rede
ORDER BY esc.ano, mun.nome, esc.rede;
```


Qualquer dúvida, entre em contato pelo email: [leidiane.santos@ifap.edu.br]
