# Rotas e Autenticação no Next.js - Site de Notícias

## Estrutura de Pastas e Rotas

O Next.js usa um sistema de arquivos para criar rotas automaticamente. Cada arquivo `page.tsx` dentro de `app` se torna uma rota acessível.

```
app/
├── (public)/              # Rotas públicas
│   ├── page.tsx           # Rota: /
│   ├── categoria/
│   │   └── [id]/
│   │       └── page.tsx   # Rota: /categoria/[id]
│   └── noticia/
│       └── [id]/
│           └── page.tsx   # Rota: /noticia/[id]
├── admin/                 # Rotas protegidas
│   ├── layout.tsx         # Layout para área admin
│   ├── page.tsx           # Rota: /admin
│   ├── categorias/
│   │   └── page.tsx       # Rota: /admin/categorias
│   └── noticias/
│       └── page.tsx       # Rota: /admin/noticias
├── auth/
│   └── login/
│       └── page.tsx       # Rota: /auth/login
└── api/                   # Rotas da API
```

### Explicando os Arquivos Especiais

- **page.tsx**: Define uma rota acessível
- **layout.tsx**: Define um layout que envolve as páginas
- **loading.tsx**: Página de carregamento
- **error.tsx**: Página de erro
- **not-found.tsx**: Página 404

## Configurando a Autenticação

### 1. Instalação das Dependências

```bash
npm install next-auth bcryptjs --legacy-peer-deps
npm install next-transpile-modules --save-dev --legacy-peer-deps
```

### 2. Configuração do Provider

```tsx
// contexts/AuthProvider.tsx
'use client'

import { SessionProvider } from "@auth/react"

export function AuthProvider({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>
}
```

### 3. Configuração da Autenticação

```typescript
// /src/auth.config.ts
import type { AuthOptions } from "next-auth"
import CredentialsProvider from "next-auth/providers/credentials"
import { prisma } from "@/lib/prisma"
import bcrypt from "bcryptjs"

declare module "next-auth" {
  interface Session {
    user: {
      id: string
      name?: string | null
      email?: string | null
    }
  }
}

declare module "next-auth/jwt" {
  interface JWT {
    id: string
  }
}

export const authConfig: AuthOptions = {
  providers: [
    CredentialsProvider({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Senha", type: "password" },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null
        }

        const user = await prisma.usuario.findUnique({
          where: { email: credentials.email },
        })

        if (!user) return null

        const isValid = await bcrypt.compare(credentials.password, user.senha)

        if (!isValid) return null

        return {
          id: user.id.toString(),
          email: user.email,
          name: user.nome,
        }
      },
    }),
  ],
  pages: {
    signIn: "/auth/login",
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id as string
        token.email = user.email
      }
      return token
    },
    async session({ session, token }) {
      if (token && session.user) {
        session.user.id = token.id as string
      }
      return session
    },
  },
}
```

### 4. Proteção de Rotas com Middleware

```typescript
// middleware.ts
import { withAuth } from "next-auth/middleware"

export default withAuth({
  callbacks: {
    authorized: ({ req, token }) => {
      const isAuth = !!token
      const isAdminPage = req.nextUrl.pathname.startsWith("/admin")
      if (isAdminPage) return isAuth
      return true
    }
  }
})

export const config = {
  matcher: ["/admin/:path*"]
}
```

## Implementação das Rotas

### 1. Layout Base da Aplicação

```tsx
// app/layout.tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { AuthProvider } from '@/contexts/AuthProvider'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'Site de Notícias',
  description: 'Portal de notícias criado com Next.js',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="pt-BR">
      <body className={inter.className}>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  )
}
```

### 2. Página de Login

```tsx
// app/auth/login/page.tsx
'use client'

import { signIn } from "next-auth/react"
import { useRouter } from "next/navigation"
import { useState } from "react"

export default function LoginPage() {
  const router = useRouter()
  const [error, setError] = useState<string | null>(null)

  async function onSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault()
    const formData = new FormData(event.currentTarget)
    const email = formData.get("email")
    const password = formData.get("password")

    try {
      const result = await signIn("credentials", {
        email,
        password,
        redirect: false
      })

      if (result?.error) {
        setError("Credenciais inválidas")
        return
      }

      router.push("/admin")
      router.refresh()
    } catch (error) {
      setError("Erro ao fazer login")
    }
  }

  return (
    <div className="min-h-screen flex items-center justify-center">
      <form onSubmit={onSubmit} className="space-y-4 w-full max-w-md">
        <h1 className="text-2xl font-bold text-center">Login</h1>
        
        {error && (
          <div className="bg-red-100 text-red-600 p-3 rounded">
            {error}
          </div>
        )}

        <div>
          <label htmlFor="email" className="block text-sm font-medium">
            Email
          </label>
          <input
            type="email"
            name="email"
            required
            className="mt-1 block w-full rounded border p-2"
          />
        </div>

        <div>
          <label htmlFor="password" className="block text-sm font-medium">
            Senha
          </label>
          <input
            type="password"
            name="password"
            required
            className="mt-1 block w-full rounded border p-2"
          />
        </div>

        <button
          type="submit"
          className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
        >
          Entrar
        </button>
      </form>
    </div>
  )
}
```

### 3. Layout da Área Administrativa

```tsx
// app/admin/layout.tsx
import { getServerSession } from "next-auth"
import { redirect } from "next/navigation"
import AdminSidebar from "./Sidebar"
import { authConfig } from "@/auth.config"

export default async function AdminLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getServerSession(authConfig)

  if (!session) {
    redirect("/auth/login")
  }

  return (
    <div className="flex min-h-screen">
      <AdminSidebar />
      <main className="flex-1 p-8">{children}</main>
    </div>
  )
}

```

### 4. Barra Lateral Administrativa

```tsx
// components/admin/Sidebar.tsx
"use client"

import Link from "next/link"
import { usePathname } from "next/navigation"
import { signOut } from "next-auth/react"

export default function AdminSidebar() {
  const pathname = usePathname()

  const links = [
    { href: "/admin", label: "Dashboard" },
    { href: "/admin/noticias", label: "Notícias" },
    { href: "/admin/categorias", label: "Categorias" },
  ]

  return (
    <aside className="w-64 bg-gray-800 text-white p-6">
      <nav className="space-y-4">
        {links.map((link) => (
          <Link
            key={link.href}
            href={link.href}
            className={`block p-2 rounded ${
              pathname === link.href ? "bg-gray-700" : ""
            }`}
          >
            {link.label}
          </Link>
        ))}

        <button
          onClick={() => signOut()}
          className="w-full text-left p-2 text-red-400 hover:bg-gray-700 rounded"
        >
          Sair
        </button>
      </nav>
    </aside>
  )
}
```

### Em Rotas da API

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth"
import { authConfig } from "@/auth.config"

const handler = NextAuth(authConfig)

export { handler as GET, handler as POST }
```

```typescript
// app/api/admin/noticias/route.ts
import { getServerSession } from "next-auth"
import { NextResponse } from "next/server"
import { authConfig } from "@/auth.config"

export async function POST(request: Request) {
  const session = await getServerSession(authConfig)

  if (!session) {
    return new NextResponse("Unauthorized", { status: 401 })
  }

  // Lógica da rota aqui
}
```

## Conceitos Importantes

1. **App Router**: Sistema de roteamento baseado em arquivos do Next.js 14
2. **Server Components**: Componentes que rodam no servidor (padrão no Next.js 14)
3. **Client Components**: Componentes que rodam no cliente (marcados com 'use client')
4. **Middleware**: Código que roda antes das requisições para proteger rotas
5. **Autenticação**: Sistema que verifica a identidade do usuário

## Fluxo de Autenticação

1. Usuário acessa a página de login
2. Insere credenciais (email e senha)
3. Sistema valida as credenciais contra o banco de dados
4. Se válido, cria uma sessão e redireciona para área administrativa
5. Middleware protege rotas administrativas verificando a sessão
6. Logout destroi a sessão e redireciona para login
