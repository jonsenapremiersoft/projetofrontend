# Rotas Públicas do Site de Notícias

## 1. Página Inicial (Home)
```tsx
// app/page.tsx
import { prisma } from "@/lib/prisma"
import NoticiaCard from "@/components/NoticiaCard"
import CategoryNav from "@/components/CategoryNav"

export default async function Home() {
  const noticias = await prisma.noticia.findMany({
    take: 10,
    orderBy: {
      dataPublicacao: 'desc'
    },
    include: {
      categoria: true,
      autor: true
    }
  })

  const categorias = await prisma.categoria.findMany()

  return (
    <main className="container mx-auto px-4">
      <CategoryNav categorias={categorias} />
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mt-6">
        {noticias.map((noticia) => (
          <NoticiaCard key={noticia.id} noticia={noticia} />
        ))}
      </div>
    </main>
  )
}
```

## 2. Componentes Necessários

### NoticiaCard
```tsx
// components/NoticiaCard.tsx
import Link from 'next/link'
import { formatDate } from '@/utils/format'
import type { Noticia, Categoria, Usuario } from '@prisma/client'

type NoticiaComRelacoes = Noticia & {
  categoria: Categoria
  autor: Usuario
}

export default function NoticiaCard({ noticia }: { noticia: NoticiaComRelacoes }) {
  return (
    <div className="border rounded-lg overflow-hidden shadow-sm hover:shadow-md transition-shadow">
      <img
        src={`https://picsum.photos/seed/${noticia.id}/400/200`}
        alt={noticia.titulo}
        className="w-full h-48 object-cover"
      />
      <div className="p-4">
        <Link 
          href={`/noticia/${noticia.id}`}
          className="text-xl font-semibold hover:text-blue-600"
        >
          {noticia.titulo}
        </Link>
        
        <div className="mt-2 text-sm text-gray-600">
          <Link 
            href={`/categoria/${noticia.categoriaId}`}
            className="text-blue-500 hover:underline"
          >
            {noticia.categoria.nome}
          </Link>
          <span className="mx-2">•</span>
          {formatDate(noticia.dataPublicacao)}
        </div>
      </div>
    </div>
  )
}
```

### CategoryNav
```tsx
// components/CategoryNav.tsx
import Link from 'next/link'
import type { Categoria } from '@prisma/client'

export default function CategoryNav({ categorias }: { categorias: Categoria[] }) {
  return (
    <nav className="flex gap-4 overflow-x-auto py-4">
      {categorias.map((categoria) => (
        <Link
          key={categoria.id}
          href={`/categoria/${categoria.id}`}
          className="px-4 py-2 rounded-full bg-gray-100 hover:bg-gray-200 whitespace-nowrap"
        >
          {categoria.nome}
        </Link>
      ))}
    </nav>
  )
}
```

## 3. Página de Categoria
```tsx
// app/categoria/[id]/page.tsx
import { prisma } from "@/lib/prisma"
import NoticiaCard from "@/components/NoticiaCard"
import { notFound } from "next/navigation"

interface Props {
  params: {
    id: string
  }
}

export default async function CategoriaPage({ params }: Props) {
  const categoria = await prisma.categoria.findUnique({
    where: { id: parseInt(params.id) }
  })

  if (!categoria) {
    notFound()
  }

  const noticias = await prisma.noticia.findMany({
    where: {
      categoriaId: parseInt(params.id)
    },
    include: {
      categoria: true,
      autor: true
    },
    orderBy: {
      dataPublicacao: 'desc'
    }
  })

  return (
    <main className="container mx-auto px-4">
      <h1 className="text-3xl font-bold mb-6">{categoria.nome}</h1>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {noticias.map((noticia) => (
          <NoticiaCard key={noticia.id} noticia={noticia} />
        ))}
      </div>
    </main>
  )
}
```

## 4. Página de Notícia
```tsx
// app/noticia/[id]/page.tsx
import { prisma } from "@/lib/prisma"
import { formatDate } from "@/utils/format"
import Link from "next/link"
import { notFound } from "next/navigation"

interface Props {
  params: {
    id: string
  }
}

export default async function NoticiaPage({ params }: Props) {
  const noticia = await prisma.noticia.findUnique({
    where: { id: parseInt(params.id) },
    include: {
      categoria: true,
      autor: true
    }
  })

  if (!noticia) {
    notFound()
  }

  return (
    <main className="container mx-auto px-4 py-8">
      <article className="max-w-4xl mx-auto">
        <h1 className="text-4xl font-bold mb-4">{noticia.titulo}</h1>
        
        <div className="mb-6 text-gray-600">
          <Link 
            href={`/categoria/${noticia.categoriaId}`}
            className="text-blue-500 hover:underline"
          >
            {noticia.categoria.nome}
          </Link>
          <span className="mx-2">•</span>
          {formatDate(noticia.dataPublicacao)}
          <span className="mx-2">•</span>
          Por {noticia.autor.nome}
        </div>

        <img
          src={`https://picsum.photos/seed/${noticia.id}/800/400`}
          alt={noticia.titulo}
          className="w-full h-[400px] object-cover rounded-lg mb-6"
        />

        <div className="prose max-w-none">
          {noticia.conteudo}
        </div>
      </article>
    </main>
  )
}
```

## 5. Utilitários

### Format
```typescript
// src/utils/format.ts
export function formatDate(date: Date) {
  return new Intl.DateTimeFormat('pt-BR', {
    day: '2-digit',
    month: 'short',
    year: 'numeric'
  }).format(new Date(date))
}
```
