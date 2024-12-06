# Configuração do Prisma e Migrations

## 1. Configuração Inicial

```bash
# Instalar dependências
npm install prisma @prisma/client

# Inicializar Prisma com SQLite
npx prisma init --datasource-provider sqlite
```

## 2. Criar Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env.DATABASE_URL
}

model Usuario {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  senha     String
  nome      String
  noticias  Noticia[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Categoria {
  id        Int       @id @default(autoincrement())
  nome      String    @unique
  noticias  Noticia[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Noticia {
  id            Int       @id @default(autoincrement())
  titulo        String
  conteudo      String
  dataPublicacao DateTime @default(now())
  autorId       Int
  autor         Usuario   @relation(fields: [autorId], references: [id])
  categoriaId   Int
  categoria     Categoria @relation(fields: [categoriaId], references: [id])
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}
```

## 3. Configurar Ambiente

```env
# .env
DATABASE_URL="file:./dev.db"
```

## 4. Criar e Aplicar Migration

```bash
# Criar migration
npx prisma migrate dev --name init

# Este comando:
# 1. Salva a migration em prisma/migrations
# 2. Aplica a migration no banco
# 3. Gera Prisma Client
```

## 5. Seed Inicial (Dados de Teste)

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'
import bcrypt from 'bcryptjs'

const prisma = new PrismaClient()

async function main() {
  // Criar usuário admin
  const admin = await prisma.usuario.create({
    data: {
      email: 'admin@example.com',
      senha: await bcrypt.hash('123456', 10),
      nome: 'Administrador'
    }
  })

  // Criar categorias
  const categorias = await prisma.categoria.createMany({
    data: [
      { nome: 'Tecnologia' },
      { nome: 'Esportes' },
      { nome: 'Política' }
    ]
  })

  // Criar notícia exemplo
  const noticia = await prisma.noticia.create({
    data: {
      titulo: 'Primeira Notícia',
      conteudo: 'Conteúdo da primeira notícia',
      autorId: admin.id,
      categoriaId: 1
    }
  })
}

main()
  .catch((e) => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

## 6. Configurar Script de Seed

```json
// package.json
{
  "scripts": {
    ////// adicionar essa linha abaixo ao scripts:///
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  },
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

## 7. Executar Seed

```/prisma/tsconfig.json```

```json
{
  "compilerOptions": {
    "target": "es2019",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true
  }
}
```

```bash
npm install -D ts-node
npx prisma db seed
```

## 8. Comandos Úteis

```bash
# Visualizar banco de dados
npx prisma studio

# Redefinir banco de dados
npx prisma migrate reset

# Verificar status das migrations
npx prisma migrate status

# Atualizar cliente após mudanças
npx prisma generate
```
