# vercel-deploy-multi-author

Template para projetos com **várias contas GitHub a contribuir** mas **apenas 1 conta Vercel (Hobby plan)**.

## O problema

- O plano Vercel **Hobby** rejeita deploys (incluindo via CLI) quando o autor do commit não tem acesso ao projeto Vercel.
- Se a equipa tem 3 pessoas no GitHub (ex. `dev-a`, `dev-b`, `dev-c`) mas só 1 conta Vercel (ex. `owner@example.com`), os pushes das outras 2 contas dão **"Blocked"** no Vercel.
- A Git Integration nativa do Vercel não resolve — verifica o autor do commit no remote, e esse autor continua a ser o da conta GitHub real.

## A solução

1. **Desligar a Git Integration do Vercel** no projeto (`Settings → Git → Disconnect`).
2. **Fazer deploy via GitHub Actions** com o `VERCEL_TOKEN` da conta dona.
3. Dentro do runner, antes de chamar o Vercel, fazer `git commit --amend` para reescrever o autor do HEAD para a conta dona. **O amend é só local no runner**, não afeta o remote nem o histórico real do GitHub.
4. Cada pessoa continua a commitar com a identidade real dela — o remote vê os commits genuínos, e o Vercel vê o autor reescrito.

## Como usar

Ver [`SETUP.md`](SETUP.md) para os passos manuais (criar token, configurar secrets, desconectar Git Integration).

Ver [`CLAUDE.md`](CLAUDE.md) se quiseres pedir ao Claude Code para replicar isto noutro projeto.

## Ficheiros

- [`.github/workflows/vercel-deploy.yml`](.github/workflows/vercel-deploy.yml) — workflow template (substitui `OWNER_EMAIL_HERE` e `OWNER_NAME_HERE`)
- [`SETUP.md`](SETUP.md) — passos manuais para configurar o projeto
- [`CLAUDE.md`](CLAUDE.md) — instruções para o Claude Code replicar isto
- [`project-CLAUDE-snippet.md`](project-CLAUDE-snippet.md) — bloco a colar no `CLAUDE.md` de cada projeto para proteger o setup
