# CRUD: Categorias

## Página de entrada

```javascript
// app/admin/categorias/page.tsx
import { prisma } from "@/lib/prisma"
import { AddCategoryButton } from "./AddCategoryButton"
import { CategoryList } from "./CategoryList"

export default async function AdminCategorias() {
  const categorias = await prisma.categoria.findMany({
    orderBy: { nome: "asc" },
  })

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Categorias</h1>
        <AddCategoryButton />
      </div>
      <CategoryList categorias={categorias} />
    </div>
  )
}
```

## AddCategoryButton

```javascript
// app/admin/categorias/AddCategoryButton.tsx
"use client"
import { useState } from "react"
import { Button } from "@/components/ui/button"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"
import { CategoryForm } from "./CategoryForm"

export function AddCategoryButton() {
  const [open, setOpen] = useState(false)

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Adicionar Categoria</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Nova Categoria</DialogTitle>
        </DialogHeader>
        <CategoryForm onSuccess={() => setOpen(false)} />
      </DialogContent>
    </Dialog>
  )
}
```

## CategoryForm

```javascript
// app/admin/categorias/CategoryForm.tsx
"use client"
import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { useRouter } from "next/navigation"

export function CategoryForm({
  categoria,
  onSuccess,
}: {
  categoria?: { id: number; nome: string }
  onSuccess?: () => void
}) {
  const [loading, setLoading] = useState(false)
  const router = useRouter()

  async function onSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault()
    setLoading(true)

    const formData = new FormData(e.currentTarget)
    const nome = formData.get("nome")

    try {
      if (categoria) {
        await fetch(`/api/admin/categorias/${categoria.id}`, {
          method: "PUT",
          body: JSON.stringify({ nome }),
        })
      } else {
        await fetch("/api/admin/categorias", {
          method: "POST",
          body: JSON.stringify({ nome }),
        })
      }
      router.refresh()
      onSuccess?.()
    } catch (error) {
      console.error("Erro ao salvar categoria:", error)
    } finally {
      setLoading(false)
    }
  }

  return (
    <form onSubmit={onSubmit} className="space-y-4">
      <Input
        name="nome"
        defaultValue={categoria?.nome}
        placeholder="Nome da categoria"
        required
      />
      <Button disabled={loading}>{loading ? "Salvando..." : "Salvar"}</Button>
    </form>
  )
}
```

## CategoryList

```javascript
// app/admin/categorias/CategoryList.tsx
"use client"
import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Card } from "@/components/ui/card"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog"
import { CategoryForm } from "./CategoryForm"
import { useRouter } from "next/navigation"

type Categoria = {
  id: number
  nome: string
}

export function CategoryList({ categorias }: { categorias: Categoria[] }) {
  const [editingCategory, setEditingCategory] = useState<Categoria | null>(null)
  const router = useRouter()

  async function handleDelete(id: number) {
    if (!confirm("Tem certeza que deseja excluir esta categoria?")) return

    try {
      await fetch(`/api/admin/categorias/${id}`, {
        method: "DELETE",
      })
      router.refresh()
    } catch (error) {
      console.error("Erro ao excluir categoria:", error)
    }
  }

  return (
    <>
      <div className="space-y-4">
        {categorias.map((categoria) => (
          <Card
            key={categoria.id}
            className="p-4 flex justify-between items-center"
          >
            <span>{categoria.nome}</span>
            <div className="space-x-2">
              <Button
                variant="outline"
                onClick={() => setEditingCategory(categoria)}
              >
                Editar
              </Button>
              <Button
                variant="destructive"
                onClick={() => handleDelete(categoria.id)}
              >
                Excluir
              </Button>
            </div>
          </Card>
        ))}
      </div>

      <Dialog
        open={!!editingCategory}
        onOpenChange={() => setEditingCategory(null)}
      >
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Editar Categoria</DialogTitle>
          </DialogHeader>
          {editingCategory && (
            <CategoryForm
              categoria={editingCategory}
              onSuccess={() => setEditingCategory(null)}
            />
          )}
        </DialogContent>
      </Dialog>
    </>
  )
}
```

## Rotas da API (src/app/api/admin/categorias)

### Rota principal: apenas cria nova categoria

```javascript
// src/app/api/admin/categorias/route.ts
import { prisma } from "@/lib/prisma"
import { NextResponse } from "next/server"
import { getServerSession } from "next-auth"
import { authConfig } from "@/auth.config"

export async function POST(request: Request) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    const { nome } = await request.json()
    const categoria = await prisma.categoria.create({
      data: { nome },
    })
    return NextResponse.json(categoria)
  } catch (error) {
    return NextResponse.json(
      { error: "Erro ao criar categoria" },
      { status: 500 }
    )
  }
}
```

### Rota dinâmica: Edita (PUT) e Deleta uma categoria pelo ID (src/app/api/admin/categorias/[id])

```javascript
// src/app/api/admin/categorias/[id]/route.ts
import { prisma } from "@/lib/prisma"
import { NextResponse } from "next/server"
import { getServerSession } from "next-auth"
import { authConfig } from "@/auth.config"

export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    const { nome } = await request.json()
    const categoria = await prisma.categoria.update({
      where: { id: parseInt(params.id) },
      data: { nome },
    })
    return NextResponse.json(categoria)
  } catch (error) {
    return NextResponse.json(
      { error: "Erro ao atualizar categoria" },
      { status: 500 }
    )
  }
}

export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession(authConfig)
  if (!session) return new NextResponse("Unauthorized", { status: 401 })

  try {
    await prisma.categoria.delete({
      where: { id: parseInt(params.id) },
    })
    return new NextResponse(null, { status: 204 })
  } catch (error) {
    return NextResponse.json(
      { error: "Erro ao excluir categoria" },
      { status: 500 }
    )
  }
}
```


