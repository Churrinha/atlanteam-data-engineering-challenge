# Documentação do Projeto: Pipeline de Dados NYC Yellow Taxi

## 1. Objetivo do Projeto

Este notebook documenta todas as etapas do pipeline de dados desenvolvido para o desafio técnico da vaga de Engenheiro de Dados Júnior/Pleno na Atlanteam.

O projeto tem como foco estruturar um pipeline confiável, escalável e interpretável baseado nos dados públicos do NYC Yellow Taxi Trip Records, aplicando as boas práticas da arquitetura em camadas (Bronze, Silver, Gold).

---

## 2. Fonte de Dados

- Dataset: Yellow Taxi Trip Data – NYC Open Data
- Período: Janeiro a Abril de 2023
- Fonte oficial: [https://data.cityofnewyork.us/Transportation/2023-Yellow-Taxi-Trip-Data/4b4i-vvec](https://data.cityofnewyork.us/Transportation/2023-Yellow-Taxi-Trip-Data/4b4i-vvec)
- Formato: JSON via API (com limite de 50.000 linhas por request)

---

## 3. Arquitetura Utilizada

O pipeline segue o padrão de arquitetura de camadas recomendado para engenharia de dados:

### Bronze
- Ingestão direta da API Socrata
- Paginada (limite de 50k registros por chamada)
- Armazenamento como Delta Table: `bronze_yellow_tripdata_2023`

### Silver
- Limpeza dos dados:
  - Passageiros > 0
  - Distância > 0
  - Tarifa > 0
- Conversão de tipos (ex: datas para timestamp)
- Armazenamento como Delta Table: `silver_yellow_tripdata_2023`

### Gold
- Criação de indicadores analíticos por mês
- Armazenamento das visões analíticas como tabelas Delta

---

## 4. Estratégia de Ingestão

A ingestão dos dados foi feita diretamente via API da NYC Open Data, utilizando paginação para contornar o limite de 50.000 linhas por requisição.

Cada lote foi:
- Convertido para DataFrame Spark
- Salvo incrementalmente em uma tabela Delta

Tabela resultante:
- Nome: `bronze_yellow_tripdata_2023`
- Volume: mais de 12,7 milhões de registros brutos

---

## 5. Tratamento e Validações (Camada Silver)

### Regras aplicadas:
- `passenger_count > 0`
- `trip_distance > 0`
- `fare_amount > 0`
- Conversão segura via `try_cast`
- Conversão de datas para o tipo `timestamp`

Tabela resultante:
- Nome: `silver_yellow_tripdata_2023`
- Volume final: 11.885.195 registros

---

## 6. Indicadores Gerados (Camada Gold)

As análises da camada Gold foram agrupadas por mês, resultando nas seguintes visões:

| Tabela                        | Descrição                                      |
|------------------------------|-----------------------------------------------|
| `gold_qtd_corridas_mes`      | Total de corridas por mês                     |
| `gold_media_tarifa_mes`      | Valor médio de tarifa por mês                 |
| `gold_receita_total_mes`     | Receita total (soma de fare_amount) por mês  |
| `gold_distancia_media_mes`   | Distância média percorrida por mês           |
| `gold_media_passageiros`     | Média geral de passageiros por corrida        |

---

## 7. Visualização Integrada

Um gráfico consolidado foi criado utilizando `matplotlib`, reunindo:

- Barras: Quantidade de corridas por mês
- Linha laranja: Receita total
- Linhas adicionais:
  - Tarifa média
  - Média de passageiros por corrida
  - Distância média

O gráfico utiliza dois eixos Y para balancear escalas diferentes em uma mesma visualização.

---

## 8. Inferências Relevantes

Com base nas análises extraídas:

- **Março** teve o maior volume de corridas e a maior receita total.
- **Fevereiro** teve o menor volume de corridas (provavelmente devido a menos dias úteis).
- A **tarifa média aumentou** consistentemente de janeiro (18,55) até abril (19,59).
- A **distância média das corridas também cresceu** mês a mês.
- A **média de passageiros por corrida se manteve constante em 1,39**, sugerindo viagens individuais.

Esses dados sugerem crescimento progressivo na demanda e no valor das corridas no primeiro quadrimestre de 2023.

---

## 9. Volume Final de Dados

- **Camada Silver:** 11.885.195 registros
- **Camadas Gold:** 5 tabelas agregadas mensais

---

## 10. Próximos Passos e Evoluções

- Criação de dashboards em Databricks SQL ou Power BI
- Inclusão de clusterização por zonas geográficas
- Agendamento do pipeline com Auto Loader e tarefas
- Enriquecimento com dados de clima ou eventos locais

---

## 11. Autor do Projeto

**Sérgio Ruffo**  
Coordenador de Dados e Analytics  
[LinkedIn](https://www.linkedin.com/in/sergio-ruffo/)  
