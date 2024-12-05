# Introdução ao Prisma, SQLite
## O que é Prisma?

https://www.prisma.io/

Prisma é um ORM (Object-Relational Mapping) moderno para Node.js e TypeScript. Ele serve como uma camada de abstração entre seu código e o banco de dados, oferecendo:

- **Type Safety**: Integração perfeita com TypeScript, fornecendo autocompletar e verificação de tipos
- **Schema Intuitivo**: Uma maneira declarativa de definir modelos de dados e relacionamentos
- **Migrations Automáticas**: Gerenciamento simplificado de alterações no banco de dados
- **Query Builder**: API intuitiva para consultas complexas
- **Studio Visual**: Interface gráfica para visualizar e editar dados

## Por que SQLite?

https://www.sqlite.org/

SQLite é um banco de dados SQL leve, que:
- Não requer servidor separado
- Armazena dados em um único arquivo
- Perfeito para desenvolvimento e aplicações menores
- Excelente para aprendizado e prototipação
- Zero configuração necessária

## Explicando o Passo a Passo da Nossa Aplicação

### 1. Configuração do Projeto
```bash
npx create-next-app@latest meu-app-prisma
```
Este comando cria um novo projeto Next.js com todas as configurações modernas necessárias. Selecionamos:
- **TypeScript**: Para tipagem estática e melhor tooling
- **Tailwind**: Framework CSS utility-first para estilização rápida
- **App Router**: Nova arquitetura do Next.js com melhor performance

### 2. Instalação do Prisma
```bash
npm install prisma --save-dev
npm install @prisma/client
```
- `prisma`: CLI e ferramentas de desenvolvimento
- `@prisma/client`: Cliente para interagir com o banco de dados

### 3. Inicialização do Prisma
```bash
npx prisma init --datasource-provider sqlite
```
Este comando:
- Cria a pasta `prisma`
- Gera o arquivo `schema.prisma`
- Configura o `.env`
- Especifica SQLite como banco de dados

### 4. Schema do Prisma
```prisma
model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  completed   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```
Cada linha tem um propósito:
- `@id`: Marca como chave primária
- `@default`: Define valores padrão
- `@updatedAt`: Atualiza timestamp automaticamente

### 5. Cliente Prisma

```lib/prisma.ts```

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = global as unknown as { prisma: PrismaClient }
export const prisma = globalForPrisma.prisma || new PrismaClient()
if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```
Este arquivo:
- Cria uma instância global do PrismaClient
- Evita múltiplas conexões durante desenvolvimento
- Mantém uma conexão eficiente com o banco

### 6. API Routes

```app/api/todos/route.ts```

```typescript
// GET: Busca todas as tarefas
export async function GET() {
  const todos = await prisma.todo.findMany({
    orderBy: { createdAt: 'desc' }
  })
  return NextResponse.json(todos)
}

// POST: Cria uma nova tarefa
export async function POST(request: Request) {
  const json = await request.json()
  const todo = await prisma.todo.create({
    data: {
      title: json.title,
    }
  })
  return NextResponse.json(todo)
}
```
Estas rotas:
- Usam o App Router do Next.js
- Implementam endpoints RESTful
- Interagem diretamente com o Prisma

### 7. Interface do Usuário

```app/page.tsx```

```typescript
"use client"

import { useState, useEffect } from "react"

interface Todo {
  id: number
  title: string
  completed: boolean
}

export default function Home() {
  const [todos, setTodos] = useState<Todo[]>([])
  const [newTodo, setNewTodo] = useState("")

  useEffect(() => {
    fetchTodos()
  }, [])

  const fetchTodos = async () => {
    const response = await fetch("/api/todos")
    const data = await response.json()
    setTodos(data)
  }

  const addTodo = async (e: React.FormEvent) => {
    e.preventDefault()
    if (!newTodo.trim()) return

    await fetch("/api/todos", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: newTodo }),
    })

    setNewTodo("")
    fetchTodos()
  }

  return (
    <main className="max-w-4xl mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">Lista de Tarefas</h1>

      <form onSubmit={addTodo} className="mb-4">
        <input
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          className="border p-2 rounded mr-2"
          placeholder="Nova tarefa..."
        />
        <button
          type="submit"
          className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
        >
          Adicionar
        </button>
      </form>

      <ul className="space-y-2">
        {todos.map((todo) => (
          <li
            key={todo.id}
            className="p-2 bg-gray-100 rounded flex justify-between items-center"
          >
            <span>{todo.title}</span>
          </li>
        ))}
      </ul>
    </main>
  )
}
```
A interface:
- Usa hooks do React para estado e efeitos
- Implementa formulário controlado
- Faz requisições à API
- Usa Tailwind para estilização


