# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sobre o Projeto

Repositório de prompts versionados para agentes de IA. Prompts são tratados como software: versionados com SemVer, testados automaticamente e revisados via PR. Documentação e comunicação do projeto são em **português brasileiro**.

## Comandos

```bash
# Setup do ambiente Python (necessário apenas para testes)
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Executar todos os testes
pytest tests/ -v

# Executar um teste específico
pytest tests/test_prompts.py::test_prompt_structure -v
```

## Arquitetura

### Estrutura de um prompt

Cada prompt vive em `agent-name/vX.Y.Z/prompt.yaml` com esta estrutura YAML:

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

Campos obrigatórios: `id`, `version`, `template`, `input_variables`. Variáveis no template usam `{nome}` (Python format string).

### Registry

`registry.yaml` é o registro central. Toda alteração de prompt precisa atualizar este arquivo. Cada entrada contém `description`, `current_version`, `path` e `model`.

### Testes

Arquivo `prompt.tests.yaml` na mesma pasta do `prompt.yaml`:

```yaml
cases:
  - name: identificador_snake_case
    inputs:
      variavel1: "valor"
    expect_contains:
      - "texto esperado no prompt renderizado"
```

Os testes são **puramente estáticos** — renderizam o template com os inputs e verificam se as strings esperadas existem no resultado. Nenhum LLM é executado.

### Validações executadas pelo pytest

1. Sintaxe do `registry.yaml`
2. Validade de todos os YAML referenciados
3. Existência física dos arquivos de prompt
4. Campos obrigatórios nos prompts
5. Variáveis do template coerentes com `input_variables`
6. Sintaxe de format string válida
7. Estrutura dos casos de teste
8. Renderização com dados de teste

### Tipos de agente

- **agent** — interage diretamente com humano (ponto de entrada)
- **subagent** — invocado por orchestrator, entrada/saída estruturada
- **orchestrator** — coordena execução sequencial de subagents
- **skill** — capacidade de produzir/ler tipo específico de arquivo
- **tool** — função chamada pelo agent durante execução

### Pipeline multi-agente (pre-sales)

O diretório `agent-discovery-pre-sales/` contém um pipeline de 5 agentes + orchestrator pendente. Os agentes se comunicam via blocos de dados com contratos definidos em `agent-discovery-pre-sales/Inventory.md`. Ao modificar qualquer agente desse pipeline, verificar os contratos de interface (blocos `[DISCOVERY_APROVADO]`, `[DISCOVERY_SUMMARY]`, `[PRICING_SUMMARY]`, `[PHASES_PLAN]`, `[PROPOSAL_METADATA]`).

## Convenções

- IDs de prompt em kebab-case: `agent-code-reviewer`
- Variáveis em snake_case: `changes_summary`
- Versionamento SemVer em pastas separadas: `v1.0.0/`, `v1.0.1/`
- Cada prompt tem `README.md` com contexto, uso e changelog
- Commits seguem Conventional Commits (`feat:`, `fix:`, `docs:`, etc.)

## Checklist para novo prompt

1. Criar pasta `agent-nome/v1.0.0/`
2. Criar `prompt.yaml` com estrutura padrão
3. Criar `prompt.tests.yaml` com casos de teste
4. Criar `agent-nome/README.md`
5. Adicionar entrada no `registry.yaml`
6. Executar `pytest tests/ -v`
