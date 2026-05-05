# 🤖 aicommit — Gerador de Commits com IA Local

Gera mensagens de commit seguindo o padrão [Conventional Commits](https://www.conventionalcommits.org/) usando modelos de linguagem rodando localmente via [Ollama](https://ollama.com). Nenhum dado sai da sua máquina.

---

## Pré-requisitos

- [Ollama](https://ollama.com) instalado e rodando
- Ao menos um dos modelos abaixo baixado
- Git configurado no projeto

### Modelos suportados

| Modelo | Qualidade | Velocidade | Recomendado para |
|---|---|---|---|
| `qwen2.5-coder:7b` | ⭐⭐⭐⭐ | Médio | Uso geral — **melhor opção** |
| `qwen2.5-coder:1.5b` | ⭐⭐⭐ | Rápido | Commits simples do dia a dia |
| `llama3.1` | ⭐⭐⭐ | Médio | Alternativa sólida |
| `gemma3.4b` | ⭐⭐ | Rápido | Não recomendado — tende a alucinar |

Para baixar um modelo:
```bash
ollama pull qwen2.5-coder:7b
```

---

## Instalação do Script

### 1. Crie a pasta bin (se não existir)
```bash
mkdir -p ~/bin
```

### 2. Crie o arquivo do script
```bash
nano ~/bin/aicommit
```

### 3. Cole o conteúdo do script
```bash
#!/bin/bash

MODEL="${AICOMMIT_MODEL:-qwen2.5-coder:7b}"

DIFF=$(git diff --cached)

if [ -z "$DIFF" ]; then
  echo "❌ Nenhum arquivo em stage. Use: git add <arquivos>"
  exit 1
fi

PROMPT="Generate a git commit message following Conventional Commits.

FORMAT: <type>(<scope>): <description>
[optional body only if needed]

TYPES: feat|fix|docs|style|refactor|perf|test|chore|ci|build
RULES:
- imperative mood, lowercase, no period, max 72 chars
- scope is optional
- output ONLY the commit message, nothing else

EXAMPLES:
feat(auth): add Google OAuth2 login
fix(api): handle null user response
chore: bump dependencies

DIFF:
$DIFF

COMMIT MESSAGE:"

MSG=$(echo "$PROMPT" | ollama run "$MODEL")

echo ""
echo "📝 Commit gerado:"
echo "─────────────────"
echo "$MSG"
echo "─────────────────"
echo ""
read -p "✅ Usar esse commit? (s/N) " CONFIRM

case "$CONFIRM" in
  s|S)
    git commit -m "$MSG"
    ;;
  *)
    echo "❌ Commit cancelado."
    ;;
esac
```

Salve: `Ctrl+O` → `Enter` → `Ctrl+X`

### 4. Torne o script executável
```bash
chmod +x ~/bin/aicommit
```

### 5. Adicione ao PATH

**macOS (zsh):**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Linux (bash):**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Como usar

```bash
# Dentro de qualquer repositório git
git add .
aicommit
```

O script vai:
1. Capturar o diff dos arquivos em stage
2. Enviar para o modelo local via Ollama
3. Exibir a mensagem gerada
4. Perguntar se deseja confirmar o commit

---

## Trocar o modelo sem editar o script

```bash
# Usar um modelo diferente pontualmente
AICOMMIT_MODEL=qwen2.5-coder:1.5b aicommit

# Definir um modelo padrão permanente (macOS)
echo 'export AICOMMIT_MODEL=qwen2.5-coder:7b' >> ~/.zshrc
source ~/.zshrc
```

---

## O Prompt

O prompt foi otimizado para modelos pequenos rodando localmente. As decisões de design:

- **Inglês** — modelos open source performam melhor em inglês
- **Formato rígido** — evita que o modelo adicione explicações ou texto extra
- **Exemplos concretos** — modelos pequenos seguem exemplos melhor que instruções abstratas
- **`COMMIT MESSAGE:` no final** — força o modelo a responder diretamente sem preâmbulo

```
Generate a git commit message following Conventional Commits.

FORMAT: <type>(<scope>): <description>
[optional body only if needed]

TYPES: feat|fix|docs|style|refactor|perf|test|chore|ci|build
RULES:
- imperative mood, lowercase, no period, max 72 chars
- scope is optional
- output ONLY the commit message, nothing else

EXAMPLES:
feat(auth): add Google OAuth2 login
fix(api): handle null user response
chore: bump dependencies

DIFF:
{DIFF}

COMMIT MESSAGE:
```

---

## Conventional Commits — Referência rápida

| Tipo | Quando usar |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Só documentação |
| `style` | Formatação, sem mudança de lógica |
| `refactor` | Reestruturação de código sem feat/fix |
| `perf` | Melhoria de performance |
| `test` | Adição ou correção de testes |
| `chore` | Build, dependências, configuração |
| `ci` | Configuração de CI/CD |

### Exemplos reais

```
feat(auth): add OAuth2 login with Google
fix(api): handle null response on user endpoint
refactor(db): extract query builder to separate module
chore: update eslint to v9
docs(readme): add Docker setup instructions
feat!: drop support for Node 14
```

O `!` após o tipo indica **breaking change**.

---

## Limitações

- **Arquivos vazios ou com pouco conteúdo** → o modelo não tem contexto suficiente e pode alucinar. Nesses casos, escreva o commit manualmente.
- **Diffs muito grandes** → pode exceder o contexto do modelo. Prefira commits menores e atômicos.
- **Qualidade depende do modelo** → `qwen2.5-coder:7b` entende diffs melhor por ter sido treinado em código.

---

## Alternativas sem script

**Direto no terminal:**
```bash
git diff --cached | ollama run qwen2.5-coder:7b "Generate a commit message following Conventional Commits for this diff. Output ONLY the message."
```

**Extensões de editor:**
- VS Code: [Ollama Commit](https://marketplace.visualstudio.com/search?term=ollama+commit&target=VSCode)
- Cursor / Zed: suporte nativo a modelos locais