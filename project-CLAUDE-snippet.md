# Bloco para o CLAUDE.md do projeto

Cola este bloco no `CLAUDE.md` do projeto-alvo (ou cria um novo se não existir). Protege o setup de deploy de futuras sessões do Claude Code.

---

## Deploys

**Deploys são 100% automáticos via GitHub Actions** (`.github/workflows/vercel-deploy.yml`):

- Push para `main` → deploy de produção no Vercel
- Abrir/atualizar PR → deploy de preview, URL comentado no PR

### O que NÃO fazer

- **Não correr `vercel deploy` local.** O workflow trata do deploy com o `VERCEL_TOKEN` da conta oficial Vercel.
- **Não fazer `git commit --amend` para mudar o autor.** Cada pessoa comita com a identidade Git real dela. O amend acontece automaticamente dentro do workflow, nunca no repo.
- **Não reativar a Git Integration do Vercel.** Foi desconectada de propósito (ver `Settings → Git` no projeto Vercel). Reativar causa "Blocked" deploys em cada push (Vercel Hobby rejeita porque o autor do commit não tem acesso). Os deploys passam apenas pelo workflow do GitHub Actions.
- **Não remover o step "Rewrite commit author for Vercel Hobby" do workflow.** Sem ele o Vercel bloqueia o deploy (o plano Hobby verifica o autor do commit mesmo em deploys via CLI). O amend é apenas local no runner, não afeta o remote nem o histórico real.

### Setup novo colaborador

1. `git clone <repo-url>`
2. Confirmar `git config user.email` — usa o email real do teu GitHub.
3. `npm install`
4. Criar branch, commitar, push, abrir PR. O preview aparece nos comentários.
