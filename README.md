# Site de Notícias: Requisitos Funcionais

## Visão Geral do Projeto
Este projeto é um site de notícias que possui duas áreas principais:
1. Área Pública: onde os visitantes podem ler as notícias
2. Área Administrativa: onde os administradores podem gerenciar o conteúdo

## 1. Área Pública

### 1.1 Página Inicial (Home)
- **Objetivo**: Apresentar as notícias mais recentes e permitir navegação por categorias
- **Funcionalidades**:
  - Exibir grid de cards de notícias com:
    - Título da notícia
    - Resumo do conteúdo
    - Imagem ilustrativa (via API pública)
    - Data de publicação
    - Categoria
  - Menu de navegação com lista de categorias
  - Paginação das notícias (10 por página)
  - Header com logo e link para login administrativo
  - Footer com informações do projeto

### 1.2 Página de Categoria
- **Objetivo**: Mostrar notícias de uma categoria específica
- **Funcionalidades**:
  - Banner com nome da categoria selecionada
  - Lista de notícias filtradas pela categoria
  - Mesma estrutura de cards da página inicial
  - Paginação específica para a categoria

### 1.3 Página de Notícia Individual
- **Objetivo**: Exibir o conteúdo completo de uma notícia
- **Funcionalidades**:
  - Título completo da notícia
  - Imagem principal (via API pública)
  - Conteúdo completo formatado
  - Informações do autor
  - Data de publicação
  - Link para a categoria
  - Navegação para voltar à listagem

## 2. Área Administrativa

### 2.1 Sistema de Autenticação
- **Objetivo**: Controlar acesso à área administrativa
- **Funcionalidades**:
  - Página de login com:
    - Campo de email
    - Campo de senha
    - Botão de entrar
    - Validação de campos
  - Sistema de sessão para manter usuário logado
  - Proteção das rotas administrativas
  - Botão de logout

### 2.2 Dashboard Administrativo
- **Objetivo**: Central de controle para administradores
- **Funcionalidades**:
  - Menu lateral com navegação administrativa
  - Resumo de notícias publicadas
  - Acesso rápido às principais funções
  - Indicador de usuário logado

### 2.3 Gerenciamento de Categorias
- **Objetivo**: Permitir controle das categorias de notícias
- **Funcionalidades**:
  - Listagem de categorias existentes
  - Formulário para adicionar nova categoria:
    - Nome da categoria
    - Validação de nome único
  - Opção de editar categoria existente
  - Opção de excluir categoria (com verificação de notícias vinculadas)

### 2.4 Gerenciamento de Notícias
- **Objetivo**: Permitir criação e edição de notícias
- **Funcionalidades**:
  - Listagem de notícias com:
    - Título
    - Categoria
    - Data de publicação
    - Status
    - Opções de edição/exclusão
  - Formulário de criação/edição:
    - Campo de título
    - Editor de conteúdo
    - Seleção de categoria
    - Preview da notícia
  - Filtros na listagem:
    - Por categoria
    - Por data
    - Por status

## 3. Regras de Negócio Importantes

### 3.1 Notícias
- Toda notícia deve ter uma categoria associada
- Notícias não podem ser publicadas sem título ou conteúdo
- A data de publicação é gerada automaticamente
- As imagens serão obtidas de API pública, não haverá upload

### 3.2 Categorias
- Não é possível ter duas categorias com o mesmo nome
- Uma categoria só pode ser excluída se não houver notícias vinculadas
- Deve existir pelo menos uma categoria no sistema

### 3.3 Autenticação
- Apenas usuários autenticados podem acessar área administrativa
- Sessões expiram após 24 horas
- Senhas devem ser armazenadas de forma segura (hash)

## 4. Experiência do Usuário

### 4.1 Interface Pública
- Design responsivo para todas as telas
- Navegação intuitiva entre categorias
- Carregamento otimizado das imagens
- Feedback visual durante carregamentos

### 4.2 Interface Administrativa
- Layout organizado e profissional
- Feedback claro para todas as ações
- Confirmações para ações destrutivas
- Mensagens de erro e sucesso claras

## 5. Integrações

### 5.1 Banco de Dados (Prisma + SQLite)
- Armazenamento de:
  - Usuários administrativos
  - Categorias
  - Notícias

### 5.2 API de Imagens
- Integração com serviço de imagens placeholder
- Fallback para imagens indisponíveis
