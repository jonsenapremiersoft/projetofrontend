# Criação e Estrutura do Projeto - Site de Notícias

## 1. Criação do Projeto

### 1.1 Requisitos do Sistema
- Node.js (versão 18 ou superior)

### 1.2 Comandos Iniciais
Abra o terminal e execute:

```bash
# Criar o projeto Next.js com as configurações necessárias
npx create-next-app@latest projeto-noticias --typescript --tailwind --app --use-npm

# Responda 'Yes' para:
# ✔ Would you like to use TypeScript? 
# ✔ Would you like to use Tailwind CSS? 
# ✔ Would you like to use the App Router? 
# ✔ Would you like to customize the default import alias (@/*)? 

# Entrar na pasta do projeto
cd projeto-noticias

# Instalar dependências adicionais necessárias
npm install @prisma/client prisma @auth/nextjs bcryptjs
npm install @types/bcryptjs --save-dev
```

## 2. Estrutura de Pastas

### 2.1 Estrutura Base
Vamos organizar nosso projeto com a seguinte estrutura:

```
src/
├── app/                    # Páginas e rotas da aplicação
├── components/             # Componentes reutilizáveis
├── lib/                    # Utilitários e configurações
├── contexts/              # Contextos do React
├── types/                 # Tipos TypeScript
└── utils/                 # Funções utilitárias
```

### 2.2 Detalhamento da Estrutura

#### 2.2.1 Pasta `app`
```
app/
├── (public)/              # Grupo de rotas públicas
│   ├── page.tsx           # Página inicial
│   ├── categoria/
│   │   └── [id]/
│   │       └── page.tsx   # Página de categoria específica
│   └── noticia/
│       └── [id]/
│           └── page.tsx   # Página de notícia individual
├── admin/                 # Área administrativa
│   ├── layout.tsx         # Layout administrativo
│   ├── page.tsx           # Dashboard
│   ├── categorias/
│   │   └── page.tsx       # Gerenciamento de categorias
│   └── noticias/
│       └── page.tsx       # Gerenciamento de notícias
├── api/                   # Rotas da API
│   ├── auth/
│   │   └── [...nextauth]
│   ├── categorias/
│   └── noticias/
├── login/
│   └── page.tsx           # Página de login
└── layout.tsx             # Layout global
```

#### 2.2.2 Pasta `components`
```
components/
├── admin/                 # Componentes da área administrativa
│   ├── CategoryForm.tsx
│   ├── NewsForm.tsx
│   └── Sidebar.tsx
├── public/               # Componentes da área pública
│   ├── NewsCard.tsx
│   ├── NewsList.tsx
│   └── CategoryNav.tsx
└── shared/              # Componentes compartilhados
    ├── Header.tsx
    ├── Footer.tsx
    └── Loading.tsx
```

#### 2.2.3 Pasta `lib`
```
lib/
├── prisma.ts            # Cliente Prisma
├── auth.ts              # Configurações de autenticação
└── constants.ts         # Constantes do projeto
```

#### 2.2.4 Pasta `contexts`
```
contexts/
└── AuthContext.tsx      # Contexto de autenticação
```

#### 2.2.5 Pasta `types`
```
types/
└── index.ts            # Tipos globais da aplicação
```

#### 2.2.6 Pasta `utils`
```
utils/
├── api.ts              # Funções de chamadas API
├── date.ts             # Funções de formatação de data
└── validation.ts       # Funções de validação
```

## 3. Configurações Iniciais

### 3.1 Configurar Prisma
Crie o arquivo `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env.DATABASE_URL
}

// Modelos serão adicionados posteriormente
```

### 3.2 Configurar Variáveis de Ambiente
Crie o arquivo `.env`:

```env
DATABASE_URL="file:./dev.db"
NEXTAUTH_SECRET="seu-secret-aqui"
```

### 3.3 Configurar Aliases
Verifique se o arquivo `tsconfig.json` tem as seguintes configurações:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## 4. Explicação da Estrutura

### 4.1 Por que esta estrutura?
- **Separação de Responsabilidades**: Cada pasta tem um propósito específico
- **Escalabilidade**: Fácil de adicionar novos recursos
- **Organização**: Código bem organizado e fácil de encontrar
- **Reutilização**: Componentes e utilitários compartilhados
- **Manutenção**: Facilita a manutenção e debugging

### 4.2 Grupos de Rotas
- **(public)**: Agrupa todas as rotas públicas
- **admin**: Área administrativa protegida
- **api**: Endpoints da API
- **login**: Autenticação separada

### 4.3 Separação de Componentes
- **admin**: Componentes exclusivos da área administrativa
- **public**: Componentes da área pública
- **shared**: Componentes utilizados em ambas as áreas

## 5. Próximos Passos

1. Configurar o banco de dados com Prisma
2. Criar os modelos de dados
3. Implementar a autenticação
4. Desenvolver os componentes base
5. Criar as rotas da API

## 6. Dicas de Desenvolvimento

1. Sempre crie novos componentes em suas respectivas pastas
2. Mantenha os componentes pequenos e reutilizáveis
3. Use tipos TypeScript para melhor segurança
4. Siga os padrões de nomenclatura estabelecidos
5. Documente funções e componentes complexos
