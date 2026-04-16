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

---

### 💡 Considerações

Essa abordagem separa claramente:

* **Modelagem de domínio** → foco em integridade e relacionamento
* **Modelagem analítica** → foco em performance e usabilidade

Permitindo evolução do pipeline, reuso de dados e maior clareza para consumo por ferramentas analíticas.
