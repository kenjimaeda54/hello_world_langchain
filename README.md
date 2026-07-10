# Hello World LangChain

Primeiro contato com o ecossistema [LangChain](https://www.langchain.com/). Projeto introdutório que demonstra a construção de uma aplicação com LLMs utilizando **LangChain** para orquestração de chains e **LangSmith** para observabilidade e tracing.

## Visão Geral

O projeto consome dados biográficos de uma personalidade e utiliza um modelo de linguagem via OpenRouter para gerar automaticamente um resumo e fatos interessantes. Todo o fluxo é instrumentado com LangSmith para depuração e monitoramento.

## Arquitetura

```
[Input] → PromptTemplate → ChatOpenRouter (tencent/hy3) → [Output]
               │                      │
               └──── LangSmith Tracing ────→ smith.langchain.com
```

- **LangChain** — Framework para construção de chains com LLMs
- **LangSmith** — Plataforma de tracing e monitoramento de execuções
- **OpenRouter** — Gateway unificado para modelos de IA (provider: Tencent, modelo: `tencent/hy3`)
- **python-dotenv** — Gerenciamento de variáveis de ambiente

## Funcionalidade

O script `hello.py` processa informações sobre **Elon Musk** e gera:

1. Um resumo curto sobre a pessoa
2. Dois fatos interessantes

O fluxo é definido com um `PromptTemplate` encadeado a um modelo via operador `|` (pipe), formando uma chain LangChain executada com `chain.invoke()`.

## Pré-requisitos

- Python >= 3.13
- Conta no [OpenRouter](https://openrouter.ai/) (chave de API)
- Conta no [LangSmith](https://smith.langchain.com/) (opcional, para tracing)

## Configuração

1. Clone o repositório e crie o ambiente virtual:

```bash
python -m venv .venv
source .venv/bin/activate
```

2. Instale as dependências:

```bash
pip install -e .
```

3. Configure o arquivo `.env` na raiz do projeto:

```env
OPEN_ROUTER_API_KEY=sk-or-v1-...
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=lsv2_pt_...
LANGSMITH_PROJECT="hello"
```

> Apenas `OPEN_ROUTER_API_KEY` é obrigatória. As variáveis `LANGSMITH_*` são necessárias apenas para tracing.

## Execução

```bash
python hello.py
```

---

## Caso de Uso: Primeiro Contato com LangChain + LangSmith

Este projeto representa o primeiro contato prático com o ecossistema LangChain. O caso de uso foi simples por design: extrair informações de um texto longo e delegar a um LLM a tarefa de sintetizar e destacar pontos relevantes.

### Por que LangChain?

- **Abstração de providers**: O `ChatOpenRouter` unifica o acesso a dezenas de modelos sem mudar o código. Foi utilizado o modelo `tencent/hy3` com temperatura 0 para respostas determinísticas.
- **Composição simples**: O operador `|` (pipe) permite encadear um `PromptTemplate` diretamente a um modelo, formando uma `Runnable` que pode ser invocada com `chain.invoke()`. A sintaxe é declarativa e familiar para quem já usa pipelines em shell ou expressões funcionais.
- **Separação entre prompt e modelo**: O template é definido como um objeto reutilizável com variáveis de entrada, facilitando manutenção e versionamento.

### Por que LangSmith?

Com as variáveis de ambiente `LANGSMITH_*` configuradas, cada execução da chain é automaticamente rastreada. Isso permitiu:

- **Visualizar o fluxo completo**: Cada step da chain (prompt → LLM → output) aparece como um span no dashboard do LangSmith, com inputs, outputs e metadados.
- **Inspecionar latência e custo**: Métricas como tempo de execução e número de tokens são exibidas por execução.
- **Depurar falhas**: Em caso de erro na chamada à API do OpenRouter, o trace captura o stack trace e permite identificar a causa sem precisar reproduzir o erro manualmente.
- **Comparar execuções**: O LangSmith mantém um histórico de todas as runs, permitindo comparar respostas lado a lado ao ajustar o prompt ou temperatura.

> Dashboard disponível em [smith.langchain.com](https://smith.langchain.com)

### Aprendizados

- A integração LangChain + LangSmith é trivial — basta configurar variáveis de ambiente para que todo tracing seja automático.
- Modelos com temperatura 0 produzem respostas consistentes e previsíveis, ideais para extração e sumarização de dados.
- O ecossistema LangChain reduz o acoplamento com provedores específicos, permitindo trocar de modelo (GPT, Claude, Llama, etc.) alterando apenas o nome do modelo no código.