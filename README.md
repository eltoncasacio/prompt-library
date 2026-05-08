# Prompts

Repositório de prompts versionados para agentes de IA. Prompts são tratados como software: versionados com SemVer, testados automaticamente e revisados via PR.

---

## Índice Categorizado

### Desenvolvimento

| Agente | Descrição | Versão | Modelo | Prompt |
|--------|-----------|--------|--------|--------|
| [agent-code-reviewer](agent-code-reviewer/README.md) | Revisão rigorosa de código (segurança, performance, boas práticas) | v1.0.0 | gpt-5-nano | [prompt.yaml](agent-code-reviewer/v1.0.0/prompt.yaml) |
| [agent-pull-request-creator](agent-pull-request-creator/README.md) | Criação de pull requests profissionais e bem documentados | v1.0.1 | gpt-5-nano | [prompt.yaml](agent-pull-request-creator/v1.0.1/prompt.yaml) |
| [agent-conventional-commit-generator](ai-commit/agent-conventional-commit-generator/README.md) | Commits Conventional via LLM local (Ollama) | v1.0.0 | qwen2.5-coder:7b | [prompt.yaml](ai-commit/agent-conventional-commit-generator/v1.0.0/prompt.yaml) |

### Pré-venda

| Agente | Descrição | Versão | Modelo | Prompt |
|--------|-----------|--------|--------|--------|
| [agent-discovery-interview](pre-sales/agent-discovery-interview/README.md) | Entrevista conversacional de discovery com cliente | v1.0.0 | Sonnet 4.5 | [prompt.yaml](pre-sales/agent-discovery-interview/v1.0.0/prompt.yaml) |
| [agent-discovery-generator](pre-sales/agent-discovery-generator/README.md) | Análise estruturada de discovery em 13 seções | v1.0.0 | Sonnet 4.5 | [prompt.yaml](pre-sales/agent-discovery-generator/v1.0.0/prompt.yaml) |
| [agent-pricing](pre-sales/agent-pricing/README.md) | Precificação por módulo com faixas defensáveis | v1.1.0 | Sonnet 4.5 | [prompt.yaml](pre-sales/agent-pricing/v1.0.0/prompt.yaml) |
| [agent-phases](pre-sales/agent-phases/README.md) | Planejamento de fases de entrega sequenciais | v1.1.0 | Sonnet 4.5 | [prompt.yaml](pre-sales/agent-phases/v1.0.0/prompt.yaml) |
| [agent-proposal-generator](pre-sales/agent-proposal-generator/README.md) | Redação de proposta comercial completa | v1.1.0 | Sonnet 4.5 | [prompt.yaml](pre-sales/agent-proposal-generator/v1.0.0/prompt.yaml) |

---

## Estrutura do Repositório

```
prompt-library/
├── registry.yaml              # Registro central de agentes e versões
├── requirements.txt           # Dependências Python para testes
├── tests/
│   └── test_prompts.py        # Validação estática de todos os prompts
│
├── <categoria>/               # Categoria do agente (ex: pre-sales, ai-commit)
│   └── <agent-nome>/          # Nome do agente em kebab-case
│       ├── README.md          # Documentação do agente
│       ├── v1.0.0/            # Primeira versão (SemVer)
│       │   ├── prompt.yaml
│       │   └── prompt.tests.yaml
│       └── v1.0.1/            # Versão atualizada (se houver)
│           ├── prompt.yaml
│           └── prompt.tests.yaml
```

Cada agente pode ter múltiplas versões. O `registry.yaml` aponta para a versão atual de cada um.

### Anatomia de um prompt

Cada prompt segue a estrutura:

```yaml
_type: prompt
id: agent-nome
version: "1.0.0"
input_variables:
  - variavel1
  - variavel2
template: |
  Texto do prompt com {variavel1} e {variavel2}
```

### Registry

O arquivo [`registry.yaml`](registry.yaml) é o registro central. Mapeia cada agente para sua versão atual, caminho do prompt e modelo recomendado:

```yaml
agents:
  agent-code-reviewer:
    description: "Agente especializado em revisão rigorosa de código"
    current_version: "1.0.0"
    path: "agent-code-reviewer/v1.0.0/prompt.yaml"
    model: gpt-5-nano
```

### Versionamento

Segue [SemVer](https://semver.org/). Cada versão vive em sua própria pasta (`v1.0.0/`, `v1.0.1/`), permitindo rollback e comparação.

---

## Testes

Os testes validam estaticamente todos os prompts sem usar LLM.

### Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Executar

```bash
pytest tests/ -v
```

### O que é validado

1. **Sintaxe YAML** — todos os arquivos referenciados no registry
2. **Existência de arquivos** — todo prompt registrado deve existir no disco
3. **Estrutura do prompt** — campos obrigatórios: `id`, `version`, `template`, `input_variables`
4. **Consistência de variáveis** — variáveis usadas no template devem estar declaradas em `input_variables`
5. **Sintaxe de f-string** — formato válido para renderização
6. **Estrutura dos casos de teste** — cada caso deve ter `name`, `inputs` e `expect_contains`
7. **Renderização** — o prompt é renderizado com os inputs do caso de teste e validado contra `expect_contains`

### Criar testes para um prompt

Um caso de teste simula o uso do prompt: você fornece valores para as variáveis de entrada (`inputs`) e declara quais textos devem aparecer no prompt renderizado (`expect_contains`). O teste renderiza o template substituindo `{variavel}` pelos valores fornecidos e verifica se todas as strings esperadas estão presentes no resultado. Nenhum LLM é executado — a validação é puramente sobre o template renderizado.

Crie um arquivo `prompt.tests.yaml` na mesma pasta do `prompt.yaml`:

```yaml
cases:
  - name: identificador_unico_do_caso
    inputs:
      variavel1: "valor simulado"
      variavel2: "outro valor"
    expect_contains:
      - "texto que deve aparecer no prompt renderizado"
      - "outro texto esperado"
```

| Campo | Descrição |
|-------|-----------|
| `name` | Identificador único do caso de teste. Use snake_case descritivo. |
| `inputs` | Dicionário com valores para cada variável declarada em `input_variables` do prompt. |
| `expect_contains` | Lista de strings que devem estar presentes no prompt após renderizar o template com os inputs. |

**Como funciona:** o template `"Analise o {codigo} na linguagem {lang}"` renderizado com `inputs: {codigo: "x = 1", lang: "python"}` produz `"Analise o x = 1 na linguagem python"`. O `expect_contains` valida que strings específicas estão nesse resultado.

#### Exemplos reais

**Prompt com uma variável** (`agent-conventional-commit-generator` — variável: `diff`):

```yaml
cases:
  - name: feat_new_function
    inputs:
      diff: |
        diff --git a/src/auth/login.py b/src/auth/login.py
        +    def login_with_google(self, token):
        +        payload = self._verify_google_token(token)
    expect_contains:
      - "feat"
      - "auth"

  - name: docs_readme_update
    inputs:
      diff: |
        diff --git a/README.md b/README.md
        -A simple project.
        +A simple project for managing tasks.
        +## Installation
    expect_contains:
      - "docs"
```

O diff é inserido no template e o teste verifica que o texto renderizado contém as palavras-chave esperadas (o tipo `feat`, o escopo `auth`, etc).

**Prompt com múltiplas variáveis** (`agent-pull-request-creator` — variáveis: `changes_summary`, `files_changed`, `branch_name`, etc):

```yaml
cases:
  - name: feature_pr_complete
    inputs:
      changes_summary: "Implementação do sistema de autenticação OAuth2"
      files_changed: "src/auth/oauth.py, tests/test_oauth.py"
      issue_number: "123"
      branch_name: "feature/oauth2-auth"
      breaking_changes: "Sim - mudança na interface de autenticação"
      testing_done: "Testes unitários executados com sucesso"
    expect_contains:
      - "OAuth2"
      - "**Breaking Changes:** Sim - mudança na interface de autenticação"
      - "**Issue Relacionada:** 123"
      - "**Branch:** feature/oauth2-auth"

  - name: docs_pr_simple
    inputs:
      changes_summary: "Atualização da documentação da API"
      files_changed: "docs/api.md, README.md"
      breaking_changes: "Não"
    expect_contains:
      - "Atualização da documentação da API"
      - "docs/api.md, README.md"
      - "### Resumo"
```

Cada caso testa um cenário diferente do prompt (feature completa, PR de docs, etc). Variáveis opcionais podem ser omitidas — o teste renderiza com string vazia no lugar.

---

## Como Contribuir

1. Crie uma branch a partir de `main`
2. Adicione ou edite o prompt seguindo a estrutura existente
3. Inclua casos de teste em `prompt.tests.yaml`
4. Atualize o `registry.yaml` se necessário
5. Execute `pytest tests/ -v` para validar
6. Abra um PR
