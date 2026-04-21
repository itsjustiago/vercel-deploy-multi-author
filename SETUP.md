# Setup manual

Passos para ligar este workflow a um projeto Vercel existente.

## 1. Obter o Vercel Token

1. Login no Vercel com a conta dona (a que tem o projeto).
2. Ir a `https://vercel.com/account/tokens`.
3. Criar token: nome `github-actions-deploy`, scope `Full Account`, expiração ao gosto.
4. Copiar o token (só é mostrado uma vez).

## 2. Obter `VERCEL_ORG_ID` e `VERCEL_PROJECT_ID`

Na máquina local, dentro do projeto:

```bash
npm install -g vercel
vercel link
```

Isto cria `.vercel/project.json` com:

```json
{
  "orgId": "team_xxxxx ou user_xxxxx",
  "projectId": "prj_xxxxx"
}
```

Anotar os dois valores. **Não commitar o `.vercel/`** (já está no `.gitignore` por default).

## 3. Adicionar secrets ao repo GitHub

No repo GitHub → `Settings → Secrets and variables → Actions → New repository secret`:

| Nome | Valor |
|---|---|
| `VERCEL_TOKEN` | token do passo 1 |
| `VERCEL_ORG_ID` | orgId do passo 2 |
| `VERCEL_PROJECT_ID` | projectId do passo 2 |

## 4. Desconectar a Git Integration do Vercel

No dashboard Vercel → projeto → `Settings → Git → Disconnect`.

**Importante:** se não fizer isto, o Vercel vai tentar fazer deploy em cada push e vai bloquear os commits de autores sem acesso. O workflow GitHub Actions passa a ser a única via de deploy.

## 5. Confirmar scope das env vars do projeto

No dashboard Vercel → projeto → `Settings → Environment Variables`, confirmar que **todas** as env vars necessárias (ex. `NEXT_PUBLIC_SUPABASE_URL`, `STRIPE_SECRET_KEY`, etc.) estão marcadas para os ambientes onde forem precisas:

- **Production** — usadas pelo deploy de `main`
- **Preview** — usadas pelos deploys de PRs (commonly skipped, depois preview falha no build)
- **Development** — usadas em `vercel dev` local

Se uma var só estiver em `Production`, o `vercel pull --environment=preview` no runner não a puxa e o build rebenta com "missing env var". Para partilhar valor entre ambientes, tick nos 3 checkboxes ao criar/editar a var. Para isolar (ex. base de dados de staging separada), põe valores diferentes em cada ambiente.

## 6. Substituir placeholders no workflow

Editar `.github/workflows/vercel-deploy.yml` e trocar:

- `OWNER_EMAIL_HERE` → email da conta Vercel dona (ex. `owner@example.com`)
- `OWNER_NAME_HERE` → nome/username a usar no amend (ex. `MyCompany`)

Aparece em 2 sítios (job `preview` e job `production`).

## 7. Commit e push

```bash
git add .github/workflows/vercel-deploy.yml
git commit -m "ci: vercel deploy via github actions"
git push
```

- Abrir PR → preview URL comentado automaticamente
- Merge para `main` → deploy de produção

## Troubleshooting

**"Blocked" no deploy:** confirmar que o amend está a correr (logs do step `Rewrite commit author for Vercel Hobby`) e que o email/nome batem certo com a conta Vercel dona.

**`vercel pull` falha:** confirmar que `VERCEL_ORG_ID` e `VERCEL_PROJECT_ID` batem certo com o `.vercel/project.json` local.

**Preview não comenta no PR:** confirmar que o workflow tem `permissions: pull-requests: write`.

**Build falha com env vars em falta (ex. `NEXT_PUBLIC_SUPABASE_URL required`, `Stripe key missing`):** as env vars existem no Vercel mas só para o ambiente `Production`. Ir a `Settings → Environment Variables` e ticar também `Preview` (e `Development` se precisar). Ver passo 5.
