# Instruções para o Claude Code

Se estás a ler isto é porque o utilizador te pediu para **replicar o setup de deploy Vercel deste template noutro projeto**. Segue os passos abaixo à risca.

## Contexto

O utilizador tem equipas com **múltiplas contas GitHub** (ex. `dev-a`, `dev-b`, `dev-c`) mas **apenas 1 conta Vercel** (plano Hobby). O Vercel Hobby bloqueia deploys de commits cujo autor não tenha acesso ao projeto. A solução é:

1. Desligar a Git Integration nativa do Vercel.
2. Usar GitHub Actions com o `VERCEL_TOKEN` da conta dona.
3. Dentro do runner, reescrever o autor do HEAD via `git commit --amend` antes de chamar o Vercel CLI. Isto engana a verificação do Hobby sem afetar o remote.

## Passos que tu (Claude Code) deves executar

### 1. Recolher informação do utilizador

Antes de escrever qualquer ficheiro, pergunta ao utilizador:

- Email da conta Vercel dona (o dono do projeto Vercel).
- Nome/username a usar como autor nos commits reescritos (tipicamente o display name da conta Vercel, ex. `SomaeSeg`, `Acme`, etc.).
- Se o projeto Vercel já existe ou precisa de ser criado.

### 2. Criar o workflow

Copia o ficheiro [`.github/workflows/vercel-deploy.yml`](.github/workflows/vercel-deploy.yml) deste repo para o projeto-alvo, em `.github/workflows/vercel-deploy.yml`.

Substitui:
- `OWNER_EMAIL_HERE` → email fornecido pelo utilizador
- `OWNER_NAME_HERE` → nome fornecido pelo utilizador

Aparece em **2 sítios** (jobs `preview` e `production`). Certifica-te que substituis em ambos.

### 3. Dar instruções manuais ao utilizador

**Não tentes** fazer os passos abaixo sozinho — são ações sensíveis que dependem de UIs externas e tokens. Em vez disso, mostra uma checklist clara ao utilizador:

```
Para ativar o deploy, precisas de fazer:

1. Criar token Vercel:
   https://vercel.com/account/tokens
   Scope: Full Account
   Guardar o token.

2. Obter orgId e projectId:
   cd <projeto>
   npm install -g vercel
   vercel link
   # abre .vercel/project.json e copia orgId + projectId

3. Adicionar 3 secrets em Settings → Secrets → Actions do repo GitHub:
   - VERCEL_TOKEN
   - VERCEL_ORG_ID
   - VERCEL_PROJECT_ID

4. No dashboard Vercel, ir a Settings → Git e carregar "Disconnect".
   (Importante: sem isto, o Vercel continua a bloquear os pushes.)

5. Commit + push do novo workflow. Abrir um PR para testar o preview.
```

### 4. Adicionar bloco ao `CLAUDE.md` do projeto

Se o projeto-alvo tiver um `CLAUDE.md`, anexa o conteúdo de [`project-CLAUDE-snippet.md`](project-CLAUDE-snippet.md) deste repo. Se não tiver, cria um `CLAUDE.md` novo com esse conteúdo. Isto protege o setup de futuras sessões do Claude que podiam desfazê-lo sem saber.

### 5. NÃO fazer

- **Não** correr `vercel deploy` local para "testar". O workflow é a única via.
- **Não** fazer `git commit --amend` no repo real para mudar autores. O amend só acontece dentro do runner do GitHub Actions.
- **Não** reativar a Git Integration do Vercel.
- **Não** remover o step "Rewrite commit author for Vercel Hobby" do workflow.
- **Não** assumir que sabes o email/nome do dono — pergunta sempre.

### 6. Verificação

Depois do utilizador fazer os passos manuais e dar push, verifica:

- Actions tab no GitHub: workflow `Vercel Deploy` deve correr verde.
- Se for PR: comentário com URL de preview aparece.
- Se for push em `main`: deploy de produção no dashboard Vercel.

Se der erro "Blocked" no step de deploy, o email/nome no amend não bate certo com a conta Vercel dona — pedir ao utilizador para reconfirmar.
