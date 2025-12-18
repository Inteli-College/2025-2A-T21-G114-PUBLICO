# Documentação de Arquitetura — ProductPro React

Este documento descreve a arquitetura completa da aplicação **ProductPro React**, incluindo frontend e backend. A aplicação é um orquestrador de prompts para agentes de IA, permitindo criar, salvar e testar configurações de prompts em tempo real usando a API da OpenAI.

---

## Sumário

1. [Visão Geral](#visão-geral)
2. [Tecnologias Utilizadas](#tecnologias-utilizadas)
3. [Estrutura de Diretórios](#estrutura-de-diretórios)
4. [Frontend](#frontend)
   - [Arquitetura](#arquitetura-do-frontend)
   - [Principais Componentes](#principais-componentes)
   - [Estado da Aplicação](#estado-da-aplicação)
   - [Tipos e Interfaces](#tipos-e-interfaces-frontend)
5. [Backend](#backend)
   - [Arquitetura](#arquitetura-do-backend)
   - [Configurações](#configurações)
   - [Middlewares](#middlewares)
   - [Rotas da API](#rotas-da-api)
   - [Banco de Dados](#banco-de-dados)
   - [Tipos e Schemas](#tipos-e-schemas-backend)
6. [Fluxo de Dados](#fluxo-de-dados)
7. [Como Executar](#como-executar)
8. [Variáveis de Ambiente](#variáveis-de-ambiente)

---

## Visão Geral

O ProductPro React é uma ferramenta de **orquestração de prompts** para agentes de IA. A aplicação permite:

- **Criar prompts estruturados** com persona, situação, tom, objetivo, guardrails e contexto adicional
- **Definir análises** que o agente deve executar antes de responder
- **Configurar funções/ferramentas** disponíveis para o agente
- **Salvar configurações** no banco de dados local para reutilização
- **Testar prompts em tempo real** enviando mensagens para a API da OpenAI

A arquitetura segue o padrão **cliente-servidor**, onde:
- O **frontend** em React fornece a interface do usuário
- O **backend** em Express gerencia a persistência e comunicação com APIs externas

---

## Tecnologias Utilizadas

### Frontend
| Tecnologia | Versão | Propósito |
|------------|--------|-----------|
| React | 18.2.0 | Biblioteca de UI |
| TypeScript | 4.9.5 | Tipagem estática |
| React Icons | 4.12.0 | Ícones SVG |
| Create React App | 5.0.1 | Bundler e toolchain |

### Backend
| Tecnologia | Versão | Propósito |
|------------|--------|-----------|
| Express | 4.21.2 | Framework web |
| TypeScript | 4.9.5 | Tipagem estática |
| Zod | 3.25.76 | Validação de schemas |
| Helmet | 7.2.0 | Headers de segurança |
| Morgan | 1.10.1 | Logger HTTP |
| CORS | 2.8.5 | Cross-Origin Resource Sharing |
| Compression | 1.8.1 | Compressão gzip |
| express-rate-limit | 7.5.1 | Rate limiting |
| dotenv | 16.6.1 | Variáveis de ambiente |

---

## Estrutura de Diretórios

```
├── public/                    # Assets estáticos
├── src/                       # Código fonte do frontend
│   ├── components/            # Componentes React reutilizáveis
│   ├── hooks/                 # Custom hooks
│   ├── services/              # Serviços de API
│   ├── App.tsx                # Componente principal
│   ├── App.css                # Estilos globais
│   ├── types.ts               # Tipos TypeScript
│   └── index.tsx              # Entry point
├── server/                    # Código fonte do backend
│   ├── data/                  # Diretório do banco de dados JSON
│   ├── middleware/            # Middlewares Express
│   ├── routes/                # Rotas da API
│   ├── config.ts              # Configurações da aplicação
│   ├── db.ts                  # Operações do banco de dados
│   ├── defaultData.ts         # Dados iniciais
│   ├── server.ts              # Entry point do servidor
│   └── types.ts               # Tipos TypeScript
├── package.json               # Dependências e scripts
├── tsconfig.json              # Config TypeScript (frontend)
└── tsconfig.server.json       # Config TypeScript (backend)
```

---

## Frontend

### Arquitetura do Frontend

O frontend é uma **Single Page Application (SPA)** construída com React 18 e TypeScript. A aplicação usa uma abordagem de estado local com `useState` e `useMemo` para gerenciamento de estado.

#### Principais Características:
- **Sem bibliotecas de estado global** — todo estado é gerenciado localmente no componente `App`
- **Comunicação via Fetch API** — requisições HTTP nativas para o backend
- **Proxy configurado** — requisições para `/api` são redirecionadas para `localhost:4000`

### Principais Componentes

O componente principal `App.tsx` contém toda a lógica de orquestração de prompts e está organizado em duas abas:

#### 1. Aba "Orquestrar" (Builder)
Formulário completo para configurar prompts com os campos:

| Campo | Descrição |
|-------|-----------|
| **Nome** | Identificador da configuração para salvar no banco |
| **Persona** | Define a identidade do agente, nível de conhecimento e estilo |
| **Situação** | Cenário atual e contexto do usuário |
| **Tom** | Voz desejada (consultivo, acolhedor, objetivo, etc.) |
| **Objetivo** | Resultado esperado e critérios de sucesso |
| **Guardrails** | Limites, políticas e temas proibidos |
| **Contexto Adicional** | Dados auxiliares, documentos e exemplos |
| **Análises** | Lista de checagens/diagnósticos a executar |
| **Funções** | Ferramentas externas e APIs disponíveis |

#### 2. Aba "Teste em Tempo Real" (Tester)
Interface de chat para testar prompts enviando mensagens para o ChatGPT:

- Seleção de fonte do prompt (rascunho atual ou configuração salva)
- Controle de temperatura (0.0 a 1.0)
- Visualização de uso de tokens
- Histórico de mensagens

### Estado da Aplicação

O estado principal é gerenciado no componente `App` através de múltiplos `useState`:

```typescript
// Formulário do prompt
const [form, setForm] = useState<PromptFormState>(INITIAL_FORM);
const [analises, setAnalises] = useState<Analise[]>([]);
const [funcoes, setFuncoes] = useState<Funcao[]>([]);

// Configurações salvas
const [savedConfigs, setSavedConfigs] = useState<PromptConfig[]>([]);
const [selectedConfigId, setSelectedConfigId] = useState<string | null>(null);

// Testador
const [testerMessages, setTesterMessages] = useState<TesterMessage[]>([]);
const [testerTemperature, setTesterTemperature] = useState(0.4);
const [testerConfigSource, setTesterConfigSource] = useState<string>('draft');
```

### Tipos e Interfaces (Frontend)

Definidos em `src/types.ts`:

```typescript
// Estado do formulário de prompt
type PromptFormState = {
  persona: string;
  situacao: string;
  tom: string;
  objetivo: string;
  guardrails: string;
  contextoAdicional: string;
};

// Análise solicitada
type Analise = {
  id: string;
  titulo: string;
  objetivo: string;
};

// Parâmetro de função
type FuncaoParametro = {
  id: string;
  nome: string;
  descricao: string;
  tipo: 'texto' | 'numero' | 'data' | 'booleano' | 'json' | 'lista';
};

// Função/ferramenta disponível
type Funcao = {
  id: string;
  nome: string;
  descricao: string;
  assinatura: string;
  parametros: FuncaoParametro[];
};

// Configuração completa salva
type PromptConfig = PromptConfigPayload & {
  id: string;
  createdAt: string;
  updatedAt: string;
};
```

---

## Backend

### Arquitetura do Backend

O backend é uma **API RESTful** construída com Express e TypeScript. Principais características:

- **Banco de dados JSON** — persistência em arquivo `db.json`
- **Validação com Zod** — schemas para validar requisições
- **Rate limiting** — proteção contra abuso
- **Segurança** — Helmet para headers seguros

#### Fluxo de Inicialização:

```
1. Carrega variáveis de ambiente (dotenv)
2. Configura middlewares (helmet, cors, compression, etc.)
3. Registra rotas da API
4. Inicializa banco de dados (cria arquivo se não existir)
5. Inicia servidor na porta configurada
```

### Configurações

Definidas em `server/config.ts`:

| Variável | Padrão | Descrição |
|----------|--------|-----------|
| `PORT` | 4000 | Porta do servidor |
| `NODE_ENV` | development | Ambiente de execução |
| `CORS_ORIGINS` | localhost:3000 | Origens permitidas |
| `RATE_LIMIT_WINDOW_MS` | 15 min | Janela do rate limit |
| `RATE_LIMIT_MAX` | 100 | Máximo de requisições por janela |
| `DB_DIR` | server/data | Diretório do banco |
| `DB_FILENAME` | db.json | Nome do arquivo |
| `OPENAI_API_KEY` | — | Chave da API OpenAI |
| `OPENAI_MODEL` | gpt-4o-mini | Modelo a usar |
| `OPENAI_BASE_URL` | api.openai.com | URL da API |

### Middlewares

#### Segurança (`server.ts`)
```typescript
app.use(helmet());          // Headers de segurança
app.use(cors(corsOptions)); // CORS configurável
app.use(compression());     // Compressão gzip
app.use(globalLimiter);     // Rate limiting global
```

#### Tratamento de Erros (`middleware/errorHandler.ts`)
```typescript
// Captura erros e retorna resposta padronizada
// Em desenvolvimento: inclui stack trace
// Em produção: mensagem genérica
```

#### Not Found (`middleware/notFound.ts`)
```typescript
// Retorna 404 para rotas não encontradas
```

### Rotas da API

#### Health Check
```
GET /health
```
Retorna status do servidor, uptime e timestamp.

---

#### Prompt Configs (CRUD)
Gerenciamento de configurações de prompt salvas.

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/prompt-configs` | Lista todas as configurações |
| `GET` | `/api/prompt-configs/:id` | Busca configuração por ID |
| `POST` | `/api/prompt-configs` | Cria nova configuração |
| `PUT` | `/api/prompt-configs/:id` | Atualiza configuração |
| `DELETE` | `/api/prompt-configs/:id` | Remove configuração |

**Payload de criação/atualização:**
```json
{
  "name": "Agente de vendas v1",
  "persona": "Especialista em vendas...",
  "situacao": "Cliente está navegando...",
  "tom": "Consultivo e amigável",
  "objetivo": "Converter lead em cliente",
  "guardrails": "Não mentir sobre preços...",
  "contextoAdicional": "Promoção de fim de ano",
  "analises": [
    { "id": "a1", "titulo": "Análise de perfil", "objetivo": "..." }
  ],
  "funcoes": [
    {
      "id": "f1",
      "nome": "buscarProduto",
      "descricao": "Busca produto no catálogo",
      "assinatura": "buscarProduto(id: string)",
      "parametros": [
        { "id": "p1", "nome": "id", "descricao": "ID do produto", "tipo": "texto" }
      ]
    }
  ]
}
```

---

#### Prompt Tester
Testa prompts enviando para a API da OpenAI.

| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/api/prompt-tester` | Envia mensagem para o modelo |

**Payload:**
```json
{
  "prompt": "### Persona\nVocê é um especialista...",
  "message": "Olá, como posso ajudar?",
  "temperature": 0.4,
  "model": "gpt-4o-mini"
}
```

**Resposta:**
```json
{
  "reply": "Olá! Estou aqui para ajudar...",
  "model": "gpt-4o-mini",
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 80,
    "total_tokens": 230
  },
  "systemPrompt": "..."
}
```

---

#### Chat (Chatbot Simples)
Chatbot básico com respostas pré-definidas.

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/chat` | Histórico de mensagens (últimas 50) |
| `POST` | `/api/chat` | Envia mensagem e recebe resposta |

---

#### Content (Dados do Site)
Endpoints para conteúdo estático do site.

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/features` | Lista de features |
| `GET` | `/api/pricing` | Planos de preços |
| `GET` | `/api/faq` | Perguntas frequentes |
| `GET` | `/api/testimonials` | Depoimentos |

---

### Banco de Dados

O banco de dados é um **arquivo JSON** (`server/data/db.json`) gerenciado pelo módulo `db.ts`.

#### Schema do Banco

```typescript
type DatabaseSchema = {
  features: Feature[];
  pricing: PricingPlan[];
  faq: FAQItem[];
  testimonials: Testimonial[];
  promptAgents: PromptAgent[];
  promptConfigs: PromptConfig[];
  chats: ChatMessage[];
};
```

#### Operações Disponíveis

```typescript
// Garante que o banco existe (cria se necessário)
ensureDatabase(): Promise<void>

// Lê todo o banco
readDatabase(): Promise<DatabaseSchema>

// Escreve todo o banco
writeDatabase(db: DatabaseSchema): Promise<DatabaseSchema>

// Lê uma coleção específica
getCollection<K extends keyof DatabaseSchema>(name: K): Promise<DatabaseSchema[K]>

// Adiciona mensagens ao chat (mantém últimas 500)
appendChatMessages(messages: ChatMessage[]): Promise<ChatMessage[]>
```

### Tipos e Schemas (Backend)

#### Validação com Zod (`routes/promptConfigs.ts`)

```typescript
// Campos de texto longo (máx 4000 caracteres)
const longText = z.string().trim().max(4000).optional().default('');

// Tipos de parâmetros suportados
const supportedParamTypes = ['texto', 'numero', 'data', 'booleano', 'json', 'lista'] as const;

// Schema de análise
const analysisSchema = z.object({
  id: z.string().trim().min(2).max(80),
  titulo: longText,
  objetivo: longText
});

// Schema de parâmetro
const parameterSchema = z.object({
  id: z.string().trim().min(2).max(80),
  nome: z.string().trim().max(120).optional().default(''),
  descricao: longText,
  tipo: z.enum(supportedParamTypes).optional().default('texto')
});

// Schema de função
const functionSchema = z.object({
  id: z.string().trim().min(2).max(80),
  nome: z.string().trim().max(200).optional().default(''),
  descricao: longText,
  assinatura: z.string().trim().max(2000).optional().default(''),
  parametros: z.array(parameterSchema).max(20).optional().default([])
});

// Schema completo da configuração
const baseConfigSchema = z.object({
  name: z.string().trim().min(3).max(120),
  persona: longText,
  situacao: longText,
  tom: z.string().trim().max(500).optional().default(''),
  objetivo: longText,
  guardrails: longText,
  contextoAdicional: longText,
  analises: z.array(analysisSchema).max(50).optional().default([]),
  funcoes: z.array(functionSchema).max(50).optional().default([])
});
```

---

## Fluxo de Dados

### Criar/Atualizar Configuração

```
┌─────────────┐     POST/PUT         ┌─────────────┐
│   Frontend  │ ─────────────────▶   │   Backend   │
│  (App.tsx)  │  /api/prompt-configs │  (Express)  │
└─────────────┘                      └──────┬──────┘
                                            │
                                            ▼
                                     ┌─────────────┐
                                     │  Validação  │
                                     │    (Zod)    │
                                     └──────┬──────┘
                                            │
                                            ▼
                                     ┌─────────────┐
                                     │   db.json   │
                                     │  (persist)  │
                                     └─────────────┘
```

### Testar Prompt

```
┌─────────────┐     POST             ┌─────────────┐     POST           ┌─────────────┐
│   Frontend  │ ─────────────────▶   │   Backend   │ ────────────────▶  │   OpenAI    │
│  (Tester)   │  /api/prompt-tester  │  (Express)  │  chat/completions  │     API     │
└─────────────┘                      └─────────────┘                    └─────────────┘
      ▲                                    │                                   │
      │                                    │                                   │
      └────────────────────────────────────┴───────────────────────────────────┘
                                    Resposta do modelo
```

---

## Como Executar

### Pré-requisitos
- Node.js 18+
- npm ou yarn

### Instalação

```bash
# Instalar dependências
npm install
```

### Desenvolvimento

```bash
# Terminal 1: Iniciar o backend (porta 4000)
npm run server

# Terminal 2: Iniciar o frontend (porta 3000)
npm start
```

### Build de Produção

```bash
# Build do frontend
npm run build

# Build do backend
npm run server:build
```

### Type Check

```bash
# Verificar tipos em todo o projeto
npm run type-check
```

---

## Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
# Servidor
PORT=4000
NODE_ENV=development
CORS_ORIGINS=http://localhost:3000

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100

# Banco de Dados
DB_DIR=server/data
DB_FILENAME=db.json

# OpenAI (necessário para testar prompts)
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
OPENAI_BASE_URL=https://api.openai.com/v1/chat/completions
```

---

## Considerações Finais

### Pontos Fortes
- **Arquitetura simples e direta** — fácil de entender e modificar
- **Validação robusta** — Zod garante integridade dos dados
- **Tipagem completa** — TypeScript em todo o projeto
- **Segurança** — rate limiting, CORS e headers seguros

### Possíveis Melhorias
- Adicionar autenticação/autorização
- Migrar para banco de dados relacional (PostgreSQL)
- Implementar testes automatizados
- Adicionar WebSocket para chat em tempo real
- Containerizar com Docker

---

*Documentação gerada em Dezembro de 2024*

