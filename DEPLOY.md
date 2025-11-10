# Guia de Deploy na Vercel

## Passo 1: Clonar o Repositório Localmente

```bash
git clone https://github.com/seu-usuario/agendamento-ia-supabase.git
cd agendamento-ia-supabase
```

## Passo 2: Instalar e Testar Localmente

```bash
npm install
npm run dev
```

Acesse http://localhost:3000 para testar

## Passo 3: Configurar o Supabase

1. Acesse https://app.supabase.com
2. Crie um novo projeto
3. Vá para SQL Editor
4. Execute o SQL do arquivo `IMPLEMENTATION_GUIDE.md`
5. Copie a URL do projeto e a Anon Key

## Passo 4: Configurar Variáveis de Ambiente Localmente

Crie um arquivo `.env.local` na raíz do projeto:

```env
NEXT_PUBLIC_SUPABASE_URL=https://seu-projeto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua-chave-anon
NEXT_PUBLIC_GEMINI_API_KEY=sua-chave-gemini
NEXT_PUBLIC_ADMIN_SECRET=seu-segredo-admin
```

## Passo 5: Deploy na Vercel

### Opção A: Via Dashboard Vercel

1. Acesse https://vercel.com
2. Clique em "New Project"
3. Selecione seu repositório GitHub
4. Clique em "Import"
5. Configure as variáveis de ambiente:
   - NEXT_PUBLIC_SUPABASE_URL
   - NEXT_PUBLIC_SUPABASE_ANON_KEY
   - NEXT_PUBLIC_GEMINI_API_KEY
   - NEXT_PUBLIC_ADMIN_SECRET
6. Clique em "Deploy"

### Opção B: Via CLI

```bash
# Instalar CLI do Vercel
npm i -g vercel

# Fazer login
vercel login

# Deploy
vercel

# Configurar variáveis de ambiente
vercel env add NEXT_PUBLIC_SUPABASE_URL
vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY
vercel env add NEXT_PUBLIC_GEMINI_API_KEY
vercel env add NEXT_PUBLIC_ADMIN_SECRET

# Deploy em produção
vercel --prod
```

## Passo 6: Verificar Status do Deploy

Acesse https://seu-projeto.vercel.app para testar o site ao vivo

## Passo 7: Configurar Domínio Customizado (Opcional)

1. No dashboard da Vercel
2. Vá para "Settings"
3. Selecione "Domains"
4. Adicione seu domínio
5. Configure os DNS registros

## Troubleshooting

### Erro: "Missing environment variables"
Certifique-se de que todas as variáveis estão configuradas no Vercel.

### Erro: "Cannot connect to Supabase"
Verifique se a URL e a chave do Supabase estão corretas.

### Erro: "Gemini API error"
Certifique-se de que tem uma chave de API válida do Google Gemini.

## Monitoramento

Após o deploy:

1. Monitore os logs em Vercel Dashboard
2. Use o Sentry para rastreamento de erros (opcional)
3. Configure alertas de performance

## Próximos Passos

1. Configurar CI/CD para testes automáticos
2. Adicionar webhooks para notificações
3. Configurar backup automático do Supabase
4. Implementar monitoramento de performance
