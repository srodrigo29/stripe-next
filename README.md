
#### 01 Passo
npm install next-auth@beta
"next-auth": "^5.0.0-beta.29",

#### 02 Passo
npx auth secret

#### 03 v1 lib/auth.ts
```
import NextAuth from 'next-auth';

export const {handlers, signIn, signOut } = NextAuth({
    providers: [],
    
    pages: {
        signIn: '/auth/signin',
        signOut: '/auth/signout',
        error: '/auth/error'
    }
});
```

#### 03 v2 lib/auth.ts
```
import NextAuth from 'next-auth';
import GitHub from "next-auth/providers/github";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from './prisma';

export const {handlers, signIn, signOut } = NextAuth({
    adapter: PrismaAdapter(prisma),
    providers: [GitHub({
        clientId: process.env.GITHUB_ID,
        clientSecret: process.env.GITHUB_SECRET
    })], 
    pages: {
        signIn: '/auth/signin',
        signOut: '/auth/signout',
        error: '/auth/error'
    }
});
```

#### 04 next auth github
``` sit doc
https://next-auth.js.org/providers/github
```

``` code usado
import GitHubProvider from "next-auth/providers/github";
```

#### 05 github config
``` url
https://github.com/settings/applications
```

``` Authorization callback URL
http://localhost:3000/api/auth/callback/github
```

``` Client ID
Ov23liAml93aGJM9mZrF
```

``` Client secrets
f1f9b4c2afa21a3fb524b920a7842d0d55e4752d
```

#### 06 Router handle
* criar a pasta
```
app/auth/[...nextauth]/route.ts
```

``` conteudo
import { handlers } from "@/lib/auth";

export const { GET, POST } = handlers;
```

#### 07 Prisma Orm
npm install @prisma/client @auth/prisma-adapter
npm install prisma --save-dev
npx prisma init

#### 08 Configurar o ORM do banco no caso usamos o neon

#### 09 Config conection
```
npx prisma generate
```
* criar o arquivo lib/prisma.ts
```
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma || new PrismaClient()

if(process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

#### 10 Prisma Adapter Prisma 26:18
``` Link
https://next-auth.js.org/v3/adapters/prisma
```

* Modelo tabelas do site atualizada
* schema.prisma
```
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

// Looking for ways to speed up your queries, or scale easily with your serverless or edge functions?
// Try Prisma Accelerate: https://pris.ly/cli/accelerate-init

generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// criamos
model Donation {
  id           String        @id @default(cuid())
  amount       Int
  donarName    String // nome do doador
  donarMessage String? // mensagem do doador
  status       PaymentStatus @default(PENDING) // status do pagamento 

  userId String
  user   User   @relation("userDonations", fields: [userId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
}

// fim criamos

// This is the Prisma schema for the NextAuth.js adapter.
// It defines the models used for user accounts, sessions, and verification requests.
model Account {
  id                 String    @id @default(cuid())
  userId             String
  providerType       String
  providerId         String
  providerAccountId  String
  refreshToken       String?
  accessToken        String?
  accessTokenExpires DateTime?
  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
  user               User      @relation(fields: [userId], references: [id])

  @@unique([providerId, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  userId       String
  expires      DateTime
  sessionToken String   @unique
  accessToken  String   @unique
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  user         User     @relation(fields: [userId], references: [id])
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  accounts      Account[]
  sessions      Session[]

  // novo
  username  String?    @unique // nome de usuário
  bio       String? // biografia do usuário
  donations Donation[] @relation("userDonations")
}

model VerificationRequest {
  id         String   @id @default(cuid())
  identifier String
  token      String   @unique
  expires    DateTime
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@unique([identifier, token])
}
```
* comando finais
``` formata o shema
npx prisma format
```

``` rodando migrate
npx prisma migrate dev
```

``` prisma studio
npx prisma studio
```

#### 11 Iniciando login 39:00
