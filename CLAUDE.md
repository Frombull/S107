# S107 — memória do projeto

Projeto da disciplina **S107 - Gerência de Configuração e Evolução de Software**.
O app é descartável: o objetivo real é praticar e testar um pipeline de CI/CD no **Jenkins**.

## Stack

- Frontend: Next.js + shadcn/ui
- Backend: NestJS + Prisma
- Banco: PostgreSQL
- Infra local: Docker Compose
- E2E: Playwright
- Deploy: Vercel
- CI/CD: Jenkins

## Regras

- **Gerenciador de pacotes: somente pnpm.** Nunca usar `npm` nem `yarn`, nem gerar `package-lock.json` ou `yarn.lock`.
  - Travado por `packageManager` e `engines` no `package.json`, `engine-strict=true` no `.npmrc` e `preinstall: only-allow pnpm`.
  - No Jenkins e nos Dockerfiles, ativar via `corepack enable` para usar a versão fixada em `packageManager`.
  - Commitar o `pnpm-lock.yaml` e instalar no CI com `pnpm install --frozen-lockfile`.
- Manter o app pequeno e simples (um CRUD basta). Priorizar o que ajuda o pipeline: builds reproduzíveis, testes rápidos, imagens Docker e estágios claros.
- Documentação e textos em português, como o README.
- **Nunca adicionar `Co-Authored-By` nem qualquer atribuição ao Claude** em commits ou PRs. Mensagens de commit ficam sem esse trailer.

## Estágios previstos do pipeline

lint e type-check → testes unitários → build → testes de integração (Postgres no Compose) → E2E (Playwright) → deploy.

## Sugestões já discutidas (ainda não adotadas)

ESLint/Prettier, Jest/Vitest, Supertest, SonarQube, Trivy, Dockerfiles multi-stage, Conventional Commits, webhook do GitHub com Multibranch Pipeline, relatórios JUnit/HTML do Playwright no Jenkins.
