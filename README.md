# Configuração do Claude Code com OpenRouter

Guia prático para configurar o **Claude Code CLI** utilizando o **OpenRouter** como provedor de API, permitindo utilizar modelos gratuitos da NVIDIA e modelos avançados da DeepSeek com roteamento dinâmico pelo menor custo (`:floor`), sem necessidade de conta direta na Anthropic.

---

## Pré-requisitos

1. **Claude Code CLI** instalado globalmente:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
2. Uma conta e chave de API do [OpenRouter](https://openrouter.ai).
3. **Crédito/Limite configurado no OpenRouter:**
   * Se for utilizar modelos pagos (ex: DeepSeek), certifique-se de que a sua chave de API possui limite de crédito configurado (diferente de `$0.00`) para evitar o erro `403 Key limit exceeded`.

---

## Configuração

### 1. Arquivo Global (`~/.claude/settings.json`)

Crie ou edite o arquivo `~/.claude/settings.json` adicionando o bloco de variáveis de ambiente (`env`):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "sk-or-v1-SUA_CHAVE_OPENROUTER_AQUI",
    "ANTHROPIC_API_KEY": "",
    "ANTHROPIC_DEFAULT_FABLE_MODEL": "deepseek/deepseek-v4.1-flash-1017:floor",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek/deepseek-v4.1-flash-1017:floor",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "nvidia/nemotron-3-ultra-550b-a55b:free",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "nvidia/nemotron-3.5-super-vl:free",
    "CLAUDE_CODE_SUBAGENT_MODEL": "nvidia/nemotron-3.5-super-vl:free",
    "CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING": "1"
  }
}
```

> **Nota:** Definir `"ANTHROPIC_API_KEY": ""` é essencial para que o Claude Code utilize o cabeçalho `Bearer` do OpenRouter em vez de exigir credenciais da Anthropic.

---

## Modelos Configurados

| Função no Claude Code | Modelo Selecionado | Custo / Roteamento |
| :--- | :--- | :--- |
| **Opus / Fable** (Tarefas complexas/default) | `deepseek/deepseek-v4.1-flash-1017:floor` | Pago (Roteado ao provider mais barato) |
| **Sonnet** (Edição e código geral) | `nvidia/nemotron-3-ultra-550b-a55b:free` | 100% Gratuito |
| **Haiku / Subagentes** | `nvidia/nemotron-3.5-super-vl:free` | 100% Gratuito |

---

## Roteamento pelo Menor Preço (`:floor`)

O sufixo `:floor` força o OpenRouter a pesquisar entre todos os provedores disponíveis que hospedam o modelo e enviar a requisição estritamente para aquele com a **menor tarifa por token**, ignorando métricas de latência ou balanceamento de carga.

Caso prefira definir essa regra globalmente para todas as chaves:
1. Acesse [https://openrouter.ai/workspaces/default/routing](https://openrouter.ai/workspaces/default/routing).
2. Em **Default Provider Sort**, altere para **Price** (ou *Floor*).
3. 10 doláres fornecem 1000 requisições diárias nos modelos free.

---

## Utilização e Comandos Úteis

Se tiver sessões anteriores associadas à Anthropic, efetue o logout:
```bash
claude auth logout
```

Inicie o assistente no terminal:
```bash
claude
```

### Alternar modelos em tempo real

Dentro do console interativo do Claude Code, utilize o comando `/model`:

* **Alternar para o modelo gratuito rápido:**
  ```text
  /model nvidia/nemotron-3.5-super-vl:free
  ```

* **Alternar para o modelo gratuito ultra:**
  ```text
  /model nvidia/nemotron-3-ultra-550b-a55b:free
  ```

* **Alternar para o DeepSeek econômico com raciocínio:**
  ```text
  /model deepseek/deepseek-v4.1-flash-1017:floor
  ```

* **Ver raciocínio interno:** Pressione `Ctrl + O` durante as gerações para expandir os blocos de raciocínio (*thought tokens*) dos modelos.

---

## Resolução de Problemas

* **`403 Key limit exceeded ($0.00 of $0.00 used)`:** Ocorre quando a chave da OpenRouter foi criada com limite fixado em `$0.00`. Acesse o painel de chaves do OpenRouter e edite o campo *Credit Limit*.
* **`400 is not a valid model ID`:** O identificador do modelo não coincide com o catálogo. Certifique-se de usar o nome completo com a contagem de parâmetros (ex: `nvidia/nemotron-3-ultra-550b-a55b:free`).
* **Saldo a zero:** O OpenRouter opera em regime pré-pago; ao esgotar os créditos, as chamadas com `:floor` são interrompidas com erro `402`, mas as rotas `:free` continuam acessíveis normalmente.
