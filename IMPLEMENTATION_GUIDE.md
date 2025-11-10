# Guia de Implementação - Sistema de Agendamento IA

## Passo 1: Configurar o Banco de Dados Supabase

Acesse: https://app.supabase.com

### Executar SQL para criar tabelas

Vá para SQL Editor e execute os seguintes comandos:

```sql
-- Tabela de Especialidades
CREATE TABLE especialidades (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL UNIQUE,
  descricao TEXT,
  icone VARCHAR(50),
  criado_em TIMESTAMP DEFAULT NOW(),
  atualizado_em TIMESTAMP DEFAULT NOW()
);

-- Tabela de Profissionais
CREATE TABLE profissionais (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  telefone VARCHAR(20),
  especialidade_id UUID REFERENCES especialidades(id) ON DELETE SET NULL,
  biografia TEXT,
  foto_url VARCHAR(500),
  ativo BOOLEAN DEFAULT TRUE,
  criado_em TIMESTAMP DEFAULT NOW(),
  atualizado_em TIMESTAMP DEFAULT NOW()
);

-- Tabela de Horarios
CREATE TABLE horarios_profissionais (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  profissional_id UUID NOT NULL REFERENCES profissionais(id) ON DELETE CASCADE,
  dia_semana SMALLINT CHECK (dia_semana BETWEEN 0 AND 6),
  hora_inicio TIME NOT NULL,
  hora_fim TIME NOT NULL,
  intervalo_minutos SMALLINT DEFAULT 30,
  ativo BOOLEAN DEFAULT TRUE,
  criado_em TIMESTAMP DEFAULT NOW(),
  UNIQUE(profissional_id, dia_semana, hora_inicio)
);

-- Tabela de Clientes
CREATE TABLE clientes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  nome VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  telefone VARCHAR(20),
  criado_em TIMESTAMP DEFAULT NOW(),
  atualizado_em TIMESTAMP DEFAULT NOW()
);

-- Tabela de Agendamentos
CREATE TABLE agendamentos (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cliente_id UUID NOT NULL REFERENCES clientes(id) ON DELETE CASCADE,
  profissional_id UUID NOT NULL REFERENCES profissionais(id) ON DELETE CASCADE,
  especialidade_id UUID NOT NULL REFERENCES especialidades(id) ON DELETE SET NULL,
  data_hora TIMESTAMP NOT NULL,
  duracao_minutos SMALLINT DEFAULT 30,
  status VARCHAR(50) DEFAULT 'pendente' CHECK (status IN ('pendente', 'confirmado', 'cancelado', 'completo')),
  notas TEXT,
  criado_em TIMESTAMP DEFAULT NOW(),
  atualizado_em TIMESTAMP DEFAULT NOW(),
  UNIQUE(profissional_id, data_hora)
);

-- Criar índices para melhor performance
CREATE INDEX idx_profissionais_especialidade ON profissionais(especialidade_id);
CREATE INDEX idx_agendamentos_cliente ON agendamentos(cliente_id);
CREATE INDEX idx_agendamentos_profissional ON agendamentos(profissional_id);
CREATE INDEX idx_agendamentos_data_hora ON agendamentos(data_hora);
CREATE INDEX idx_agendamentos_status ON agendamentos(status);
```

## Passo 2: Criar os Arquivos do Projeto

### Estrutura de Diretórios

```
app/
├── api/
│   ├── profissionais/
│   │   ├── route.ts
│   │   ├── [id]/
│   │   │   └── route.ts
│   │   └─┐ horarios/
│   │       └── route.ts
│   ├── agendamentos/
│   │   ├── route.ts
│   │   └── [id]/
│   │       └── route.ts
│   └── gemini/
│       └── route.ts
├── admin/
│   ├── layout.tsx
│   ├── dashboard/
│   │   └── page.tsx
│   ├── profissionais/
│   │   └── page.tsx
│   ├── horarios/
│   │   └── page.tsx
│   ├── especialidades/
│   │   └── page.tsx
│   ├── clientes/
│   │   └── page.tsx
│   └── agendamentos/
│       └── page.tsx
├── chat/
│   ├── page.tsx
│   └── components/
│       ├── ChatBox.tsx
│       ├── Calendar.tsx
│       └── ProfessionalList.tsx
├── page.tsx
└── layout.tsx

components/
├── AdminNav.tsx
├── AdminButton.tsx
└── Loading.tsx

lib/
├── supabase.ts
├── gemini.ts
├── utils.ts
└── hooks/
    ├── useSupabase.ts
    └── useChat.ts

types/
└── index.ts
```

## Passo 3: Arquivos-Chave para Criar

### 1. lib/supabase.ts

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseKey);
```

### 2. lib/gemini.ts

```typescript
import { GoogleGenerativeAI } from '@google/generative-ai';

const genAI = new GoogleGenerativeAI(process.env.NEXT_PUBLIC_GEMINI_API_KEY!);

export const model = genAI.getGenerativeModel({ 
  model: 'gemini-pro',
  systemInstruction: 'Você é um assistente de agendamento de serviços. Ajude os clientes a agendar serviços com profissionais especialistas. Seja atencioso e prestativo.'
});
```

### 3. types/index.ts

```typescript
export interface Especialidade {
  id: string;
  nome: string;
  descricao: string;
  icone?: string;
  criado_em: string;
}

export interface Profissional {
  id: string;
  nome: string;
  email: string;
  telefone: string;
  especialidade_id: string;
  biografia?: string;
  foto_url?: string;
  ativo: boolean;
  criado_em: string;
}

export interface HorarioProfissional {
  id: string;
  profissional_id: string;
  dia_semana: number; // 0-6
  hora_inicio: string; // HH:MM
  hora_fim: string; // HH:MM
  intervalo_minutos: number;
  ativo: boolean;
}

export interface Cliente {
  id: string;
  nome: string;
  email: string;
  telefone?: string;
  criado_em: string;
}

export interface Agendamento {
  id: string;
  cliente_id: string;
  profissional_id: string;
  especialidade_id: string;
  data_hora: string; // ISO 8601
  duracao_minutos: number;
  status: 'pendente' | 'confirmado' | 'cancelado' | 'completo';
  notas?: string;
  criado_em: string;
}
```

## Passo 4: Implementar Componentes React

### app/chat/page.tsx

```typescript
'use client';

import { useState } from 'react';
import ChatBox from './components/ChatBox';
import Calendar from './components/Calendar';
import ProfessionalList from './components/ProfessionalList';
import AdminButton from '@/components/AdminButton';

export default function ChatPage() {
  const [selectedDate, setSelectedDate] = useState<Date | null>(null);
  const [availableProfessionals, setAvailableProfessionals] = useState([]);

  const handleDateSelect = (date: Date) => {
    setSelectedDate(date);
    // Buscar profissionais disponíveis
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
      <AdminButton />
      <div className="container mx-auto px-4 py-8">
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Calendário */}
          <div className="lg:col-span-1">
            <Calendar onSelectDate={handleDateSelect} />
          </div>

          {/* Chat e Profissionais */}
          <div className="lg:col-span-2 space-y-6">
            {selectedDate && (
              <>
                <ProfessionalList date={selectedDate} />
                <ChatBox selectedDate={selectedDate} />
              </>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```

## Passo 5: Deploy na Vercel

1. Conecte seu repositório GitHub à Vercel
2. Configure as variáveis de ambiente:
   - NEXT_PUBLIC_SUPABASE_URL
   - NEXT_PUBLIC_SUPABASE_ANON_KEY
   - NEXT_PUBLIC_GEMINI_API_KEY
   - NEXT_PUBLIC_ADMIN_SECRET

3. Clique em "Deploy"

## Passo 6: Testar

```bash
# Instalar dependências
npm install

# Rodar em desenvolvimento
npm run dev

# Acessar em http://localhost:3000
```

## Documentação da API

Várias endpoints serão criadas para gerenciar o sistema.

### POST /api/agendamentos
Cria um novo agendamento

### GET /api/profissionais?especialidade=id&data=2024-01-01
Lista profissionais disponíveis em uma data

### POST /api/gemini
Processa mensagens do chatbot

## Próximas Melhorias

- [ ] Autenticação de usuário
- [ ] Sistema de notificações por email
- [ ] Integração com pagamento (Stripe)
- [ ] Relatórios avançados
- [ ] App mobile
