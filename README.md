# case_bee
Case de modelagem de dados 

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

### 🌐 Uso de API Externa

Para enriquecer os dados e aumentar seu valor analítico, foi integrada uma API pública de câmbio, permitindo a conversão de valores monetários para uma moeda padrão (BRL).

A API foi utilizada para obter taxas de conversão atualizadas entre diferentes moedas, possibilitando a padronização dos valores presentes na base de pedidos. Esse enriquecimento viabiliza análises financeiras consistentes, independentemente da moeda original de cada transação.

A integração foi realizada de forma eficiente, evitando chamadas linha a linha. As taxas foram coletadas em lote, transformadas em uma tabela de apoio (lookup) e posteriormente integradas aos dados via join distribuído no Spark.

Além disso, foi necessário ajustar a direção da taxa de câmbio (invertendo os valores retornados pela API), garantindo a correta conversão para a moeda desejada. O resultado final inclui um novo campo com o valor convertido (`amount_brl`), pronto para agregações e análises.

Essa abordagem demonstra o uso de fontes externas para enriquecimento de dados, mantendo escalabilidade, rastreabilidade e consistência no pipeline.

