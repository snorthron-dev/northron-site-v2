# Northron Security — Guia de Instalação e Deploy

Este documento explica como rodar o projeto localmente e publicá-lo gratuitamente fora da plataforma Manus.

---

## Visão Geral da Arquitetura

O projeto é uma aplicação full-stack com um único repositório:

| Camada | Tecnologia | Descrição |
|---|---|---|
| **Frontend** | React 19 + Vite + Tailwind 4 | Interface do site e painel admin |
| **Backend** | Node.js + Express + tRPC | API, autenticação e regras de negócio |
| **Banco de dados** | MySQL / MariaDB | Orçamentos, leads, usuários admin |
| **Armazenamento** | Local (pasta ./uploads) | Fotos de perfil dos funcionários (sem dependência da Manus) |
| **Autenticação** | JWT + bcrypt | Login próprio com username e senha |

Em desenvolvimento, o Vite serve o frontend e o Express serve o backend na mesma porta (3000) via proxy. Em produção, o build gera arquivos estáticos que o Express também serve.

---

## Pré-requisitos

Antes de começar, instale:

- **Node.js 20+** — [nodejs.org](https://nodejs.org)
- **pnpm** — `npm install -g pnpm`
- **MySQL 8.0+** ou **MariaDB 10.5+** — [mysql.com](https://mysql.com) ou via Docker
- **Git** — [git-scm.com](https://git-scm.com)

---

## Rodando Localmente (Passo a Passo)

### 1. Clonar ou extrair o projeto

```bash
# Se recebeu o ZIP, extraia e entre na pasta:
unzip northron-security.zip
cd northron-security

# Ou clone do Git (se tiver repositório):
git clone https://github.com/seu-usuario/northron-security.git
cd northron-security
```

### 2. Instalar as dependências

```bash
pnpm install
```

### 3. Configurar o banco de dados

Crie o banco e as tabelas executando o script SQL incluído:

```bash
# Acesse o MySQL como root:
mysql -u root -p

# Dentro do MySQL, execute:
source criar-banco.sql

# Ou em uma linha:
mysql -u root -p < criar-banco.sql
```

**Alternativa com Docker** (sem instalar MySQL):

```bash
docker run -d \
  --name northron-mysql \
  -e MYSQL_ROOT_PASSWORD=suasenha \
  -e MYSQL_DATABASE=northron_security \
  -p 3306:3306 \
  mysql:8.0

# Aguarde ~10 segundos e execute o script:
docker exec -i northron-mysql mysql -uroot -psuasenha northron_security < criar-banco.sql
```

### 4. Configurar as variáveis de ambiente

Copie o arquivo de configuração modelo e renomeie para `.env`:

```bash
cp config-modelo.txt .env
```

Edite o `.env` com seus valores reais:

```
DATABASE_URL=mysql://root:suasenha@localhost:3306/northron_security
JWT_SECRET=cole-aqui-uma-chave-aleatoria-longa
NODE_ENV=development
PORT=3000
```

Para gerar uma chave JWT segura:

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### 5. Iniciar o servidor de desenvolvimento

```bash
pnpm dev
```

Acesse: **http://localhost:3000**

O painel admin fica em: **http://localhost:3000/admin/login**

**Credenciais iniciais:**
- Username: `thiago.northron.adm`
- Senha: `Adonai.01`

> **Importante:** Troque a senha no primeiro acesso pelo painel em Funcionários → seu perfil.

---

## Estrutura do Projeto

```
northron-security/
├── client/                  ← Frontend React
│   ├── src/
│   │   ├── pages/           ← Páginas do site (Home, Sobre, FAQ, etc.)
│   │   ├── pages/admin/     ← Painel administrativo
│   │   ├── components/      ← Componentes reutilizáveis
│   │   └── index.css        ← Tema preto + dourado
│   └── index.html
├── server/                  ← Backend Node.js
│   ├── routers.ts           ← Endpoints tRPC (orçamentos, admin, auth)
│   ├── db.ts                ← Helpers de banco de dados
│   └── storage.ts           ← Upload de arquivos
├── drizzle/
│   └── schema.ts            ← Definição das tabelas
├── criar-banco.sql          ← Script SQL para criar as tabelas
├── config-modelo.txt        ← Modelo do arquivo .env
└── package.json
```

---

## Gerenciando Funcionários

### Criar um novo funcionário pelo painel

1. Faça login em `/admin/login` com sua conta admin
2. Vá em **Funcionários** no menu lateral
3. Clique em **+ Novo Funcionário**
4. Preencha nome, cargo e senha temporária
5. O username é gerado automaticamente no padrão `primeironome.northron`

### Padrão de username

| Nome do funcionário | Username gerado |
|---|---|
| João Silva | `joao.northron` |
| Maria Santos | `maria.northron` |
| Segundo João | `joao2.northron` |
| Você (admin) | `thiago.northron.adm` |

### Criar funcionário diretamente no banco (alternativa)

```bash
# Gere o hash da senha:
node -e "const b=require('bcryptjs'); b.hash('SenhaTemporaria123',12).then(console.log)"

# Execute no MySQL:
INSERT INTO admin_users (username, passwordHash, nomeCompleto, cargo, ativo, primeiroAcesso)
VALUES ('joao.northron', 'HASH_GERADO_ACIMA', 'João Silva', 'adestrador', 'sim', 'sim');
```

### Níveis de acesso

| Cargo | Orçamentos | Leads | Funcionários | Configurações |
|---|---|---|---|---|
| **admin** | Ver + Editar | Ver | Criar + Editar + Remover | Sim |
| **adestrador** | Ver + Editar | Não | Não | Não |
| **recepcao** | Ver | Ver | Não | Não |

---

## Adaptações Necessárias para Rodar Fora da Manus

O projeto foi desenvolvido na plataforma Manus e usa alguns serviços dela. Para rodar de forma independente, você precisa:

### 1. Remover ou adaptar o storage.ts

O `server/storage.ts` atual usa a API de storage da Manus. Para upload de fotos de perfil independente, substitua por AWS S3 direto:

```bash
pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

Substitua o conteúdo de `server/storage.ts` por:

```typescript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: process.env.AWS_REGION || "us-east-1",
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
  // Para Cloudflare R2, adicione:
  // endpoint: process.env.AWS_ENDPOINT_URL,
});

export async function storagePut(
  relKey: string,
  data: Buffer | Uint8Array | string,
  contentType = "application/octet-stream"
): Promise<{ key: string; url: string }> {
  const bucket = process.env.AWS_S3_BUCKET!;
  await s3.send(new PutObjectCommand({
    Bucket: bucket,
    Key: relKey,
    Body: data as any,
    ContentType: contentType,
    ACL: "public-read",
  }));
  const url = `https://${bucket}.s3.${process.env.AWS_REGION}.amazonaws.com/${relKey}`;
  return { key: relKey, url };
}
```

### 2. Remover notificações da Manus (opcional)

Em `server/routers.ts`, as notificações de novos orçamentos usam `notifyOwner()` da Manus. Para desativar, remova as linhas que importam e chamam `notifyOwner`. Para substituir por email, use o pacote `nodemailer`.

### 3. Remover o OAuth da Manus (opcional)

As rotas `/api/oauth/*` e o sistema de login OAuth não são necessários se você usar apenas o login admin próprio. Você pode ignorá-las — elas não afetam o funcionamento do site.

---

## Deploy Gratuito

### Opção A: Railway (Recomendado — tudo em um lugar)

Railway hospeda o Node.js + MySQL gratuitamente no plano Hobby ($5/mês após trial).

1. Crie conta em [railway.app](https://railway.app)
2. Clique em **New Project → Deploy from GitHub**
3. Conecte seu repositório GitHub
4. Adicione um serviço MySQL: **New → Database → MySQL**
5. Configure as variáveis de ambiente no painel do Railway
6. O deploy acontece automaticamente a cada push

**Variáveis a configurar no Railway:**

```
DATABASE_URL=  (Railway preenche automaticamente ao linkar o MySQL)
JWT_SECRET=sua-chave-jwt-aqui
NODE_ENV=production
```

### Opção B: Render (Backend) + PlanetScale (MySQL)

**Backend no Render:**

1. Crie conta em [render.com](https://render.com)
2. Clique em **New → Web Service**
3. Conecte o repositório GitHub
4. Configure:
   - **Build Command:** `pnpm install && pnpm build`
   - **Start Command:** `node dist/index.js`
5. Adicione as variáveis de ambiente

**Banco no PlanetScale:**

1. Crie conta em [planetscale.com](https://planetscale.com) (plano gratuito disponível)
2. Crie um banco chamado `northron-security`
3. Copie a connection string e cole em `DATABASE_URL`
4. Execute o script SQL pelo console do PlanetScale

### Opção C: VPS (Controle Total)

Para quem quer controle total e tem um servidor VPS (DigitalOcean, Contabo, etc.):

```bash
# No servidor, instale Node.js 20 e MySQL
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs mysql-server

# Clone o projeto
git clone https://github.com/seu-usuario/northron-security.git
cd northron-security

# Configure o banco
mysql -u root -p < criar-banco.sql

# Configure o .env
cp config-modelo.txt .env
nano .env  # edite com seus valores

# Instale dependências e faça o build
pnpm install
pnpm build

# Inicie com PM2 (gerenciador de processos)
npm install -g pm2
pm2 start dist/index.js --name northron
pm2 save
pm2 startup

# Configure Nginx como proxy reverso
sudo apt install nginx
# Aponte o domínio para localhost:3000
```

---

## Build para Produção

```bash
# Gera os arquivos otimizados
pnpm build

# Inicia o servidor de produção
NODE_ENV=production node dist/index.js
```

O servidor Express serve tanto a API (`/api/*`) quanto os arquivos estáticos do frontend.

---

## Comandos Úteis

| Comando | Descrição |
|---|---|
| `pnpm dev` | Inicia o servidor de desenvolvimento |
| `pnpm build` | Gera o build de produção |
| `pnpm test` | Executa os testes automatizados |
| `pnpm db:push` | Sincroniza o schema com o banco (Drizzle) |

---

## Suporte

Em caso de dúvidas técnicas sobre o projeto, consulte a documentação das tecnologias usadas:

- [tRPC](https://trpc.io/docs) — API type-safe
- [Drizzle ORM](https://orm.drizzle.team) — Banco de dados
- [Vite](https://vitejs.dev) — Build do frontend
- [Tailwind CSS 4](https://tailwindcss.com) — Estilização
