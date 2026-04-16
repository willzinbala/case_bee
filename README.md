# 📊 Case Beecrowd

## 📌 Escopo Geral

Este projeto tem como objetivo a construção de um pipeline de dados completo, contemplando:

* **Modelagem de dados** baseada em entidades de negócio (customers, orders e events)
* **Tratamento e padronização de dados** provenientes de múltiplas fontes (CSV, JSON e JSONL)
* **Aplicação de arquitetura em camadas (Raw, Trusted e Refined)**
* **Enriquecimento de dados com API externa**, permitindo a padronização de valores monetários

O foco principal foi garantir qualidade, rastreabilidade e capacidade analítica dos dados, simulando um cenário real de engenharia de dados.

---

## 🛠️ Tecnologias Utilizadas

* **Python**
* **PySpark**
* **Apache Spark (via Google Colab)**
* **Parquet (armazenamento otimizado)**
* **API REST (Exchange Rate API)**
* **Requests (consumo de API)**

---

### 🏗️ Solução Encontrada

A solução foi construída utilizando o conceito de **arquitetura em camadas**, estruturada da seguinte forma:


## 🧩 Modelagem de Dados e Arquitetura em Camadas

A modelagem de dados foi estruturada considerando dois pilares complementares: o **modelo conceitual do domínio (ER)** e a **arquitetura em camadas (Raw, Trusted e Refined)**.

---

### 🔹 Modelo Entidade-Relacionamento (ER)

O domínio foi modelado a partir de três entidades principais:

* **Customer**
* **Order**
* **Event**

#### Relacionamentos:

* Um **Customer** pode possuir múltiplos **Orders** *(1:N)*
* Um **Customer** pode possuir múltiplos **Events** *(1:N)*

#### Estrutura conceitual:

```
Customer
---------
customer_id (PK)
name
email
country
created_at

Order
---------
order_id (PK)
customer_id (FK)
amount
currency
status
order_date

Event
---------
event_id (PK)
customer_id (FK)
event_type
event_timestamp
```

Esse modelo representa o núcleo do negócio, garantindo integridade e clareza nas relações entre os dados.

---

### 🏗️ Arquitetura em Camadas

A implementação segue o padrão de arquitetura em camadas, separando responsabilidades ao longo do pipeline:

#### 🔸 Raw

* Armazena os dados no formato original (CSV, JSON, JSONL)
* Mantém fidelidade à fonte
* Permite reprocessamento e auditoria

#### 🔸 Trusted

* Aplica padronização e tipagem
* Remove inconsistências e duplicidades
* Garante integridade das chaves e relações
* Representa o modelo mais próximo do ER

#### 🔸 Refined

* Integra diferentes entidades
* Aplica regras de negócio e enriquecimentos (ex: conversão de moeda)
* Estrutura dados para consumo analítico
* Prioriza performance e simplicidade de consulta

---

### 🔗 Integração entre Modelo e Camadas

* O **modelo ER** é refletido principalmente na camada **Trusted**, onde os dados estão normalizados e consistentes
* A camada **Refined** utiliza esse modelo como base para criar visões desnormalizadas, otimizadas para análise

Exemplos:

* `ref_orders_customers` → visão analítica de pedidos + clientes
* `ref_events_customers` → visão analítica de eventos + clientes
* `ref_orders_brl` → visão unificada de faturamento em BRL

### 🌐 Uso de API Externa - `exchangerate-api`

Para enriquecer os dados e aumentar seu valor analítico, foi integrada uma API pública de câmbio, permitindo a conversão de valores monetários para uma moeda padrão (BRL).

A API foi utilizada para obter taxas de conversão atualizadas entre diferentes moedas, possibilitando a padronização dos valores presentes na base de pedidos. Esse enriquecimento viabiliza análises financeiras consistentes, independentemente da moeda original de cada transação.

A integração foi realizada de forma eficiente, evitando chamadas linha a linha. As taxas foram coletadas em lote, transformadas em uma tabela de apoio (lookup) e posteriormente integradas aos dados via join distribuído no Spark.

Além disso, foi necessário ajustar a direção da taxa de câmbio (invertendo os valores retornados pela API), garantindo a correta conversão para a moeda desejada. O resultado final inclui um novo campo com o valor convertido (`amount_brl`), pronto para agregações e análises.


---

## 🔄 Outras Possíveis Soluções

A abordagem adotada com **notebook (.ipynb)** foi intencionalmente mais enxuta, focada em rapidez de desenvolvimento e clareza na demonstração do pipeline.

Outras abordagens possíveis incluem:

* **Modelagem em banco SQL**

  * Criação de tabelas normalizadas
  * Uso de Python para ingestão e transformação
  * Execução de queries analíticas diretamente no banco

* **Projeto completo em repositório Git**

  * Estrutura modularizada (scripts, configs, pipelines)
  * Uso de ferramentas como Airflow ou dbt
  * Versionamento e organização por camadas

Essas alternativas aumentariam robustez e organização, porém com maior complexidade de implementação.

---

## ▶️ Como Executar o Projeto

1. Baixar ou clonar o notebook (`.ipynb`)
2. Subir o arquivo no Google Colab
3. Criar a estrutura de pastas (`raw`, `trusted`, `refined`)
4. Fazer upload das 3 fontes de dados na camada **raw**:

   * `customers.csv`
   * `events.jsonl`
   * `orders.json`
5. Executar as células do notebook **em sequência**

Ao final, os dados tratados e enriquecidos estarão disponíveis nas camadas **trusted** e **refined**.

---

## 🚀 Futuras Melhorias

Para evolução do projeto em um cenário mais próximo de produção, algumas melhorias podem ser implementadas:

* **Data Quality e Observabilidade**

  * Implementação de validações automatizadas (ex: regras de consistência, integridade e completude)
  * Monitoramento de métricas como volume, nulls, duplicidade e distribuição de dados
  * Alertas para falhas ou degradação da qualidade

* **Processamento em Tempo Real**

  * Evolução do pipeline batch para streaming com Spark Structured Streaming
  * Ingestão contínua de eventos em tempo real
  * Atualizações incrementais nas camadas trusted e refined

* **Câmbio com Base Histórica**

  * Integração com APIs que fornecem taxa de câmbio por data
  * Conversão de valores baseada no `order_date`, garantindo maior precisão financeira
  * Armazenamento de histórico de taxas para reprocessamento

* **Camada Semântica (Data Mart)**

  * Modelagem em formato dimensional (Star Schema)
  * Criação de tabelas fato e dimensão (ex: fact_orders, dim_customers)
  * Definição de métricas de negócio padronizadas

* **Orquestração de Pipeline**

  * Uso de ferramentas como Apache Airflow para agendamento e controle de dependências
  * Execução automatizada e monitorada dos pipelines
  * Reprocessamento controlado em caso de falhas

* **Versionamento e Estrutura de Projeto**

  * Organização do projeto em um repositório estruturado (ex: separação por camadas e módulos)
  * Versionamento de código e pipelines
  * Uso de boas práticas de engenharia de software

* **Testes e Validações**

  * Implementação de testes unitários para transformações
  * Testes de integração entre camadas
  * Validação de contratos de dados (data contracts)

* **Performance e Escalabilidade**

  * Particionamento eficiente dos dados (ex: por data ou país)
  * Otimização de joins e uso de broadcast quando aplicável
  * Ajustes de configuração do Spark para grandes volumes

* **Consumo e Visualização**

  * Integração com ferramentas de BI (Power BI, Tableau)
  * Criação de dashboards interativos
  * Exposição de dados via APIs ou camada de serviço

* **Governança de Dados**

  * Controle de acesso e segurança
  * Catalogação de dados (ex: Data Catalog)
  * Documentação técnica e funcional das tabelas

Essas melhorias transformariam o projeto em uma solução mais robusta, escalável e aderente a ambientes reais de engenharia de dados.


---

## 📈 Considerações e Conclusões

O projeto demonstra na prática a construção de um pipeline de dados completo, desde a ingestão até a disponibilização para análise.

A utilização de arquitetura em camadas permite:

* Separação clara de responsabilidades
* Maior controle sobre qualidade dos dados
* Facilidade de manutenção e evolução

O enriquecimento com API externa adiciona valor analítico, mostrando a importância de integrar dados internos com fontes externas para geração de insights mais completos.

---

## 👨‍💻 Autor

**Willian José Nogueira**
Engenheiro de Dados
Desafio Técnico
