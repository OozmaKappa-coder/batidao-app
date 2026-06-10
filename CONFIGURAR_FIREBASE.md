# 🔥 Guia de Configuração do Firebase — Casa de Sucos Batidão

---

## PASSO 1 – Criar o Projeto no Firebase

1. Acesse https://console.firebase.google.com
2. Clique em **"Adicionar projeto"**
3. Nome: `batidao-app` → Avançar → Criar projeto

---

## PASSO 2 – Registrar o App Web

1. No painel do projeto, clique no ícone **`</>`** (Web)
2. Nome do app: `Batidão App`
3. Clique em **"Registrar app"**
4. Copie o objeto `firebaseConfig` que aparecer. Ele tem este formato:

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "batidao-app.firebaseapp.com",
  projectId:         "batidao-app",
  storageBucket:     "batidao-app.appspot.com",
  messagingSenderId: "1234567890",
  appId:             "1:1234:web:abcdef"
};
```

5. **Cole esse objeto no `index.html`**, substituindo o `firebaseConfig` de demonstração (linha ~180)

---

## PASSO 3 – Ativar Authentication

1. No menu esquerdo: **Authentication → Começar**
2. Aba **"Sign-in method"** → Ativar **"E-mail/senha"**
3. Aba **"Users"** → **"Adicionar usuário"**:

| E-mail | Senha | Perfil |
|--------|-------|--------|
| `gerente@batidao.com` | `gerente123` | Gerente |
| `atendente@batidao.com` | `atend123` | Atendente |

---

## PASSO 4 – Configurar Firestore Database

1. Menu esquerdo: **Firestore Database → Criar banco de dados**
2. Escolha **"Iniciar no modo de teste"** (libera leitura/escrita por 30 dias)
3. Escolha a região: **southamerica-east1 (São Paulo)**

### Criar coleções iniciais (opcional, o app cria automaticamente):
- `usuarios` — perfis dos usuários
- `pedidos` — pedidos em tempo real
- `produtos` — cardápio
- `clientes` — cadastro de clientes

---

## PASSO 5 – Regras de Segurança do Firestore (IMPORTANTE)

Vá em **Firestore → Regras** e substitua pelo conteúdo abaixo:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Usuários só leem/editam o próprio documento
    match /usuarios/{uid} {
      allow read, write: if request.auth.uid == uid;
    }

    // Pedidos: apenas usuários autenticados
    match /pedidos/{pedidoId} {
      allow read, write: if request.auth != null;
    }

    // Produtos: todos leem; apenas gerente escreve
    match /produtos/{produtoId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.perfil == 'gerente';
    }

    // Clientes: apenas autenticados
    match /clientes/{clienteId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## PASSO 6 – Criar o Perfil do Gerente no Firestore

Após o primeiro login como gerente, o app cria o documento automaticamente.
Mas você pode criar manualmente:

1. Firestore → Criar coleção: `usuarios`
2. ID do documento: (UID do usuário gerente — copie de Authentication → Users)
3. Campos:
   - `email`: `gerente@batidao.com` (string)
   - `perfil`: `gerente` (string)

---

## PASSO 7 – Adicionar Produtos Iniciais

No app, faça login como **gerente** e vá em **Cardápio → + Produto**.

Sugestões de categorias para começar:
- **Sucos**: Laranja, Limão, Maracujá, Abacaxi, Melancia
- **Combos**: Suco + Lanche
- **Adicionais**: Leite condensado, Granola, Mel
- **Lanches**: Sanduíche natural, Açaí

---

## PASSO 8 – Hospedar o App (opcional, gratuito)

### Opção A – Firebase Hosting
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Public dir: . (ponto)
# Single page app: não
firebase deploy
```
URL final: `https://batidao-app.web.app`

### Opção B – GitHub Pages
1. Crie repositório no GitHub
2. Suba o `index.html`
3. Settings → Pages → Branch: main

### Opção C – Netlify (arraste e solte)
1. Acesse https://netlify.com
2. Arraste a pasta `batidao-app` para a área de deploy
3. Pronto! URL gerada automaticamente.

---

## ✅ Checklist Final

- [ ] `firebaseConfig` substituído no `index.html`
- [ ] Authentication ativado com e-mail/senha
- [ ] Usuários gerente e atendente criados
- [ ] Firestore criado (região São Paulo)
- [ ] Regras de segurança configuradas
- [ ] Perfil do gerente criado no Firestore
- [ ] Produtos adicionados no cardápio
- [ ] App hospedado (opcional)
- [ ] Testado em dois celulares ao mesmo tempo ✨

---

## 📱 Como usar nos celulares das duas lojas

1. Abra o app no navegador (Chrome recomendado)
2. Menu do Chrome → **"Adicionar à tela inicial"**
3. O app aparecerá como se fosse um app nativo
4. Qualquer pedido criado em um celular aparece no outro em segundos ⚡

---

*Criado para Casa de Sucos Batidão · Sistema de Pedidos v1.0*
