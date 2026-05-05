# Versionamento de Prompts

## 🧠 Uso Geral
Prompts para utilização diária com foco em:
- produtividade  
- pesquisa  
- exploração  

> Defina se o prompt é para uso pessoal ou para equipe.

---

## 👥 Gerenciamento de Prompts (Equipe)

- Tratar prompt como **software**
- Uso de:
  - Git
  - Testes
  - PRs (Pull Requests)
  - Code Review
  - Evaluation
  - Produção

---

## ⚙️ Setup

- Clone o repositório de prompts na sua máquina local
- Crie um **link simbólico** para uso no projeto

Junto com seu projeto, crie uma pasta prompt e dentro dela crie um arquivo registry.yaml e os prompts 

> ⚠️ Não clone o repositório de prompts dentro do projeto para evitar “poluir o contexto”

---

## 📚 Estrutura do Repositório

### README.md
- Deve conter um **índice categorizado**

---

## 📄 Documentação por Prompt

Cada prompt deve possuir documentação contendo:

- Nome do Prompt  
- Link para o prompt “plain”  
- Objetivo  
- Modelos testados / utilizados  
- Observações gerais  
- Últimas atualizações  
- (Opcional) Verificação de vulnerabilidades (`Doc.md`)

---

## 🗂️ Categorização

- **Exploração e Planejamento**  
  (Análise de codebase, segurança, guidelines, novas features)

- **Documentação**  
  (Criação e manutenção de Design Docs)

- **Implementação**  
  (Testes, bugs, refatoração, LLM-as-a-judge)

- **Especialização**  
  (Go, Spring, Laravel, LangGraph)

- **Infraestrutura**  
  (Docker, k8s, Terraform, CI/CD)

- **Utils**  
  (Commits, PRs, Code Reviews)

---

## 🧠 Arquitetura de Prompts (Produto)

### Lógica Básica

- Registry (todos os prompts + versões)
- Identificação por:
  - agente
  - caso de uso
  - componente
  - subdomínio
- Versionamento (SemVer)
- Arquivo do prompt
- Metadados (podem estar no próprio arquivo)
- Casos de teste (validação)
- README com:
  - contexto
  - uso
  - changelog

---

# 🚀 Exemplo (Langfuse)

## 🧠 Fluxo

```
lead_inbound.agent.main
 ├── lead_inbound.agent.pre_attendance (opcional)
 │     └── lead_inbound.skill.collect_info
 │
 ├── lead_distribution.agent.routing
 │     ├── lead_distribution.skill.classify_lead
 │     ├── lead_distribution.skill.select_seller
 │     └── lead_distribution.skill.validate_assignment
 │
 ├── lead_distribution.skill.generate_message
 │
 └── followup.agent.scheduler
       └── followup.skill.generate_message
```

---

## 🏗️ Naming

### 🔹 Entrada do Lead

```
lead_inbound.agent.main
lead_inbound.agent.pre_attendance
lead_inbound.skill.collect_info
```

**Responsabilidades:**
- Interpretar mensagem (WhatsApp)
- Decidir pré-atendimento
- Extrair dados iniciais

---

### 🔹 Distribuição (Core)

```
lead_distribution.agent.routing
lead_distribution.skill.classify_lead
lead_distribution.skill.select_seller
lead_distribution.skill.validate_assignment
lead_distribution.skill.generate_message
```

**Responsabilidades:**
- Classificar lead
- Selecionar vendedor (round-robin + regras)
- Validar distribuição
- Gerar mensagem de handoff

---

### 🔹 Follow-up

```
followup.agent.scheduler
followup.skill.generate_message
```

**Responsabilidades:**
- Decidir reengajamento
- Gerar mensagens automáticas

---

## 🏷️ Tags (Langfuse)

| Tipo            | Tags                   |
|-----------------|------------------------|
| agent.main      | agent, orchestration   |
| agent.routing   | agent, decision        |
| skill.classify  | skill, classifier      |
| skill.select    | skill, routing         |
| skill.generate  | skill, generation      |
| followup        | agent, automation      |

---

## 🔢 Versionamento (SemVer)

Use apenas para prompts críticos:

```
lead_distribution.agent.routing:v1.0.0
lead_distribution.skill.select_seller:v1.1.0
```

---

## 🧩 Exemplo de Prompt

### lead_distribution.skill.select_seller

**Input**
```json
{
  "lead": {...},
  "sellers": [...],
  "rules": {...}
}
```

**Output**
```json
{
  "seller_id": "123",
  "reason": "round_robin_next"
}
```

---

## 🔥 Decisões de Arquitetura

### ✔ Separação clara
- agent → decide  
- skill → executa  

---

### ⚠️ Evitar
- agent fazendo tudo  
- prompts gigantes  
- lógica misturada  

---

### 🚀 Benefícios
- trocar apenas partes específicas (select_seller)
- testar isoladamente (classify_lead)
- versionar pontos críticos
