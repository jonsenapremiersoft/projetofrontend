# Admin Notícias

## Actions: para evitar repetição de código (na parte de categorias não implementamos isso)

```javascript
// src/app/admin/noticias/actions.ts
'use server'
import { prisma } from "@/lib/prisma"

export async function getNoticias() {
 return prisma.noticia.findMany({
   include: {
     categoria: true,
     autor: true
   },
   orderBy: { dataPublicacao: 'desc' }
 })
}

export async function createNoticia(data: {
 titulo: string
 conteudo: string
 categoriaId: number
 autorId: number
}) {
 return prisma.noticia.create({
   data: {
     ...data,
     dataPublicacao: new Date()
   },
   include: {
     categoria: true,
     autor: true
   }
 })
}

export async function updateNoticia(
 id: number,
 data: {
   titulo: string
   conteudo: string
   categoriaId: number
 }
) {
 return prisma.noticia.update({
   where: { id },
   data,
   include: {
     categoria: true,
     autor: true
   }
 })
}

export async function deleteNoticia(id: number) {
 return prisma.noticia.delete({
   where: { id }
 })
}
```

### Page Notícias

```javascript
// src/app/admin/noticias/page.tsx
import { prisma } from "@/lib/prisma"
import { AddNewsButton } from "./AddNewsButton"
import { NewsList } from "./NewsList"

export default async function AdminNoticias() {
  const [noticias, categorias] = await Promise.all([
    prisma.noticia.findMany({
      include: {
        categoria: true,
        autor: true
      },
      orderBy: {
        dataPublicacao: 'desc'
      }
    }),
    prisma.categoria.findMany()
  ])

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Notícias</h1>
        <AddNewsButton categorias={categorias} />
      </div>
      <NewsList noticias={noticias} categorias={categorias} />
    </div>
  )
}
```

### Components

```javascript
// src/app/admin/noticias/AddNewsButton.tsx
'use client'
import { useState } from "react"
import { Button } from "@/components/ui/button"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"
import { NewsForm } from "./NewsForm"
import type { Categoria } from "@prisma/client"

export function AddNewsButton({ categorias }: { categorias: Categoria[] }) {
  const [open, setOpen] = useState(false)

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Adicionar Notícia</Button>
      </DialogTrigger>
      <DialogContent className="max-w-3xl">
        <DialogHeader>
          <DialogTitle>Nova Notícia</DialogTitle>
        </DialogHeader>
        <NewsForm categorias={categorias} onSuccess={() => setOpen(false)} />
      </DialogContent>
    </Dialog>
  )
}
```

```javascript
// src/app/admin/noticias/NewsForm.tsx
'use client'
import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"
import { 
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"
import { useRouter } from "next/navigation"
import type { Categoria, Noticia } from "@prisma/client"

type NoticiaEdit = Noticia & {
  categoria: Categoria
}

interface NewsFormProps {
  noticia?: NoticiaEdit
  categorias: Categoria[]
  onSuccess?: () => void
}

export function NewsForm({ noticia, categorias, onSuccess }: NewsFormProps) {
  const [loading, setLoading] = useState(false)
  const router = useRouter()

  async function onSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault()
    setLoading(true)

    const formData = new FormData(e.currentTarget)
    const data = {
      titulo: formData.get('titulo'),
      conteudo: formData.get('conteudo'),
      categoriaId: parseInt(formData.get('categoriaId') as string)
    }

    try {
      if (noticia) {
        await fetch(`/api/admin/noticias/${noticia.id}`, {
          method: 'PUT',
          body: JSON.stringify(data)
        })
      } else {
        await fetch('/api/admin/noticias', {
          method: 'POST',
          body: JSON.stringify(data)
        })
      }
      router.refresh()
      onSuccess?.()
    } catch (error) {
      console.error('Erro ao salvar notícia:', error)
    } finally {
      setLoading(false)
    }
  }

  return (
    <form onSubmit={onSubmit} className="space-y-4">
      <Input
        name="titulo"
        defaultValue={noticia?.titulo}
        placeholder="Título da notícia"
        required
      />
      
      <Select name="categoriaId" defaultValue={noticia?.categoriaId.toString()}>
        <SelectTrigger>
          <SelectValue placeholder="Selecione uma categoria" />
        </SelectTrigger>
        <SelectContent>
          {categorias.map(categoria => (
            <SelectItem 
              key={categoria.id} 
              value={categoria.id.toString()}
            >
              {categoria.nome}
            </SelectItem>
          ))}
        </SelectContent>
      </Select>

      <Textarea
        name="conteudo"
        defaultValue={noticia?.conteudo}
        placeholder="Conteúdo da notícia"
        required
        className="min-h-[200px]"
      />

      <Button disabled={loading}>
        {loading ? 'Salvando...' : 'Salvar'}
      </Button>
    </form>
  )
}
```

```javascript
// src/app/admin/noticias/NewsList.tsx
'use client'
import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Card } from "@/components/ui/card"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog"
import { NewsForm } from "./NewsForm"
import { useRouter } from "next/navigation"
import type { Noticia, Categoria, Usuario } from "@prisma/client"

type NoticiaComRelacoes = Noticia & {
  categoria: Categoria
  autor: Usuario
}

interface NewsListProps {
  noticias: NoticiaComRelacoes[]
  categorias: Categoria[]
}

export function NewsList({ noticias, categorias }: NewsListProps) {
  const [editingNews, setEditingNews] = useState<NoticiaComRelacoes | null>(null)
  const router = useRouter()

  async function handleDelete(id: number) {
    if (!confirm('Tem certeza que deseja excluir esta notícia?')) return

    try {
      await fetch(`/api/admin/noticias/${id}`, {
        method: 'DELETE'
      })
      router.refresh()
    } catch (error) {
      console.error('Erro ao excluir notícia:', error)
    }
  }

  return (
    <>
      <div className="space-y-4">
        {noticias.map((noticia) => (
          <Card key={noticia.id} className="p-4">
            <div className="flex justify-between items-start">
              <div>
                <h3 className="font-medium">{noticia.titulo}</h3>
                <p className="text-sm text-gray-500 mt-1">
                  {noticia.categoria.nome} • {new Date(noticia.dataPublicacao).toLocaleDateString()}
                </p>
              </div>
              <div className="space-x-2">
                <Button
                  variant="outline"
                  onClick={() => setEditingNews(noticia)}
                >
                  Editar
                </Button>
                <Button
                  variant="destructive"
                  onClick={() => handleDelete(noticia.id)}
                >
                  Excluir
                </Button>
              </div>
            </div>
          </Card>
        ))}
      </div>

      <Dialog
        open={!!editingNews}
        onOpenChange={() => setEditingNews(null)}
      >
        <DialogContent className="max-w-3xl">
          <DialogHeader>
            <DialogTitle>Editar Notícia</DialogTitle>
          </DialogHeader>
          {editingNews && (
            <NewsForm
              noticia={editingNews}
              categorias={categorias}
              onSuccess={() => setEditingNews(null)}
            />
          )}
        </DialogContent>
      </Dialog>
    </>
  )
}
```


## Rotas

```javascript
// src/app/api/admin/noticias/route.ts
import { prisma } from "@/lib/prisma"
import { NextResponse } from "next/server"
import { getServerSession } from "next-auth"
import { authConfig } from "@/auth.config"

export async function POST(request: Request) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    const data = await request.json()
    const noticia = await prisma.noticia.create({
      data: {
        ...data,
        autorId: parseInt(session.user.id),
        dataPublicacao: new Date()
      }
    })
    return NextResponse.json(noticia)
  } catch (error) {
    return NextResponse.json({ error: "Erro ao criar notícia" }, { status: 500 })
  }
}
```

```javascript
// src/app/api/admin/noticias/[id]/route.ts
import { prisma } from "@/lib/prisma"
import { NextResponse } from "next/server"
import { getServerSession } from "next-auth"
import { authConfig } from "@/auth.config"

export async function PUT(request: Request, { params }: { params: { id: string } }) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    const data = await request.json()
    const noticia = await prisma.noticia.update({
      where: { id: parseInt(params.id) },
      data
    })
    return NextResponse.json(noticia)
  } catch (error) {
    return NextResponse.json({ error: "Erro ao atualizar notícia" }, { status: 500 })
  }
}

export async function DELETE(request: Request, { params }: { params: { id: string } }) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    await prisma.noticia.delete({
      where: { id: parseInt(params.id) }
    })
    return new NextResponse(null, { status: 204 })
  } catch (error) {
    return NextResponse.json({ error: "Erro ao excluir notícia" }, { status: 500 })
  }
}
```

