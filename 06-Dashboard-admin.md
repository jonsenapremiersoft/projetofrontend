# Dashboard Admin

## Dependências usadas

```bash
npx shadcn@latest add card
```

```javascript
// app/admin/page.tsx
import { getServerSession } from "next-auth"
import { authConfig } from "../../auth.config"
import { prisma } from "@/lib/prisma"
import { Card } from "@/components/ui/card"

export default async function AdminHome() {
 const session = await getServerSession(authConfig)
 
 // Buscar estatísticas básicas
 const stats = await prisma.$transaction([
   prisma.noticia.count(),
   prisma.categoria.count(),
   prisma.noticia.findMany({
     take: 5,
     orderBy: { dataPublicacao: 'desc' },
     include: { categoria: true }
   })
 ])

 const [totalNoticias, totalCategorias, ultimasNoticias] = stats

 return (
   <div className="space-y-6">
     <h1 className="text-3xl font-bold">Dashboard</h1>
     
     <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
       <Card className="p-6">
         <h3 className="text-lg font-medium">Total de Notícias</h3>
         <p className="text-3xl font-bold mt-2">{totalNoticias}</p>
       </Card>
       
       <Card className="p-6">
         <h3 className="text-lg font-medium">Categorias</h3>
         <p className="text-3xl font-bold mt-2">{totalCategorias}</p>
       </Card>
     </div>

     <div className="mt-8">
       <h2 className="text-xl font-bold mb-4">Últimas Notícias</h2>
       <div className="space-y-4">
         {ultimasNoticias.map((noticia) => (
           <Card key={noticia.id} className="p-4">
             <h3 className="font-medium">{noticia.titulo}</h3>
             <p className="text-sm text-gray-500 mt-1">
               {noticia.categoria.nome} • {new Date(noticia.dataPublicacao).toLocaleDateString()}
             </p>
           </Card>
         ))}
       </div>
     </div>
   </div>
 )
}
```
