# Agendamento IA com Supabase

Sistema completo de agendamento com IA Gemini, chat inteligente, calendário e painel administrativo.

## Características

- Chat inteligente com IA Gemini
- Calendário interativo para seleção de datas
- Listagem de profissionais disponíveis por especialidade e horário
- Painel administrativo completo
- Banco de dados Supabase integrado
- Hospedagem na Vercel

## Estrutura do Projeto

```
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── admin/
│   │   ├── dashboard/
│   │   ├── professionais/
│   │   ├── horarios/
│   │   ├── especialidades/
│   │   ├── clientes/
│   │   └── agendamentos/
│   ├── chat/
│   │   ├── page.tsx
│   │   └── components/
│   ├── api/
│   │   ├── profissionais/
│   │   ├── agendamentos/
│   │   ├── horarios/
│   │   └── gemini/
│   └── layout.tsx
├── components/
│   ├── Calendar/
│   ├── ChatBox/
│   ├── ProfessionalCard/
│   └── AdminNav/
├── lib/
│   ├── supabase.ts
│   ├── gemini.ts
│   └── hooks/
├── types/
│   └── index.ts
└── public/
```

## Instalação

1. Clone o repositório:
```bash
git clone https://github.com/seu-usuario/agendamento-ia-supabase.git
cd agendamento-ia-supabase
```

2. Instale as dependências:
```bash
npm install
```

3. Configure as variáveis de ambiente no arquivo `.env.local`:
```env
NEXT_PUBLIC_SUPABASE_URL=sua_url_supabase
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua_public_key
NEXT_PUBLIC_GEMINI_API_KEY=sua_gemini_api_key
```

4. Execute o servidor de desenvolvimento:
```bash
npm run dev
```

5. Acesse http://localhost:3000

## Configuração do Supabase

### Tabelas Necessárias

#### 1. Tabela: `especialidades`
```sql
CREATE TABLE especialidades (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL,
  descricao TEXT,
  criado_em TIMESTAMP DEFAULT NOW()
);
```

#### 2. Tabela: `profissionais`
```sql
CREATE TABLE profissionais (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL,
  especialidade_id UUID REFERENCES especialidades(id),
  email VARCHAR(255) UNIQUE,
  telefone VARCHAR(20),
  foto_url VARCHAR(500),
  ativo BOOLEAN DEFAULT TRUE,
  criado_em TIMESTAMP DEFAULT NOW()
);
```

#### 3. Tabela: `horarios_profissionais`
```sql
CREATE TABLE horarios_profissionais (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  profissional_id UUID REFERENCES profissionais(id) ON DELETE CASCADE,
  dia_semana INTEGER (0-6),
  hora_inicio TIME,
  hora_fim TIME,
  intervalo_minutos INTEGER DEFAULT 30,
  ativo BOOLEAN DEFAULT TRUE,
  criado_em TIMESTAMP DEFAULT NOW()
);
```

#### 4. Tabela: `clientes`
```sql
CREATE TABLE clientes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE,
  telefone VARCHAR(20),
  criado_em TIMESTAMP DEFAULT NOW()
);
```

#### 5. Tabela: `agendamentos`
```sql
CREATE TABLE agendamentos (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cliente_id UUID REFERENCES clientes(id),
  profissional_id UUID REFERENCES profissionais(id),
  especialidade_id UUID REFERENCES especialidades(id),
  data_hora TIMESTAMP NOT NULL,
  duracao_minutos INTEGER DEFAULT 30,
  status VARCHAR(50) DEFAULT 'pendente',
  notas TEXT,
  criado_em TIMESTAMP DEFAULT NOW(),
  UNIQUE(profissional_id, data_hora)
);
```

## Funcionalidades da IA

O sistema utiliza a IA Gemini para:

1. **Chat inteligente**: Conversar com o usuário sobre agendamentos
2. **Recomendação de profissionais**: Sugerir profissionais baseado em especialidade
3. **Análise de disponibilidade**: Verificar horarios disponíveis
4. **Confirmação de agendamento**: Processar e confirmar agendamentos

## Fluxo do Usuário

1. Usuário abre o chat
2. Clica em um dia no calendário
3. IA busca profissionais disponíveis naquele dia
4. Usuário seleciona um profissional
5. IA confirma o agendamento
6. Agendamento é salvo no Supabase

## Painel Administrativo

O painel administrativo permite:

- Cadastrar profissionais
- Definir horários de atendimento
- Gerenciar especialidades
- Gerenciar clientes
- Visualizar agendamentos
- Gerar relatórios

## Deploy na Vercel

1. Conecte seu repositório GitHub à Vercel
2. Configure as variáveis de ambiente na Vercel
3. Faça deploy

```bash
npm run build
npm start
```

## Tecnologias Utilizadas

- **Frontend**: React 18, Next.js 14, TypeScript
- **Backend**: Next.js API Routes
- **Banco de Dados**: Supabase (PostgreSQL)
- **IA**: Google Gemini API
- **Estilo**: Tailwind CSS
- **Hospedagem**: Vercel

## Lição de Aprendizado

Este projeto demonstra:

- Integração com IA Gemini
- Uso de Supabase para backend
- Criação de sistemas de agendamento
- Desenvolvimento full-stack com Next.js
- Deploy em producão

## Licença

MIT

## Suporte

Para suporte, crie uma issue no repositório GitHub.
