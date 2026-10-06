# S107 — memória do projeto

Projeto da disciplina **S107 - Gerência de Configuração e Evolução de Software**.
O app é descartável: o objetivo real é praticar e testar um pipeline de CI/CD no **Jenkins**.

## O app: Cathoot

O app se chama **Cathoot** (nome definido em 2026-10-06): o "Kahoot de tribunal de gato". Ideia escolhida em 2026-10-05 (o conceito original era "Tribunal dos Gatos"). Cadastra-se um gato (nome, imagem meme e o "crime" cometido) e a turma vota **Culpado** ou **Inocente**. Será usado numa votação em sala de aula na apresentação, então precisa ser divertido e funcionar bem com muita gente votando ao mesmo tempo.

Regras de negócio previstas (alvo dos testes):

- um voto por pessoa por gato;
- o veredito muda ao atingir o limite de votos;
- gato condenado vai para a "Prisão da Soneca";
- nome do gato e crime obrigatórios, imagem com URL válida.

Fluxo E2E principal: cadastrar gato → votar → ver o veredito.

## Formato: "Kahoot de Tribunal de Gato" (Cathoot)

Decidido em 2026-10-05. Ideia: simples e divertido.

1. O apresentador (painel protegido por senha simples) cria uma sessão e mostra um QR code com o código da sala.
2. A turma entra pelo QR, digita só o nome e espera no lobby.
3. O apresentador clica em "Próximo": aparece o gato com o crime, e os alunos votam.
4. O apresentador clica em "Revelar": todos veem o placar e o veredito. Depois vem o próximo gato.
5. No fim, um resumo (gato mais culpado etc.).

Estados da sessão: `LOBBY → VOTANDO → RESULTADO → VOTANDO → ... → FIM`. Só se vota em `VOTANDO`, o voto é único e imutável, o nome é único na sessão e só o apresentador avança o estado.

Decisões técnicas:

- Monorepo único (`apps/web` com Next.js; `apps/api` com NestJS virá depois), workspace pnpm na raiz.
- Deploy na Vercel com **Root Directory = `./apps/web`**; tempo real por **polling** (~1,5 s), sem WebSocket.
- Deploy só quando o código do `apps/web` muda: `ignoreCommand` em `apps/web/vercel.json` (`git diff --quiet HEAD^ HEAD -- . ':(exclude)*.md'`, roda dentro do Root Directory). O **Skip deployment** automático da Vercel não basta: arquivos fora do workspace (`apps/*`), como o `README.md` e o `CLAUDE.md` da raiz, contam como mudança global e geram deploy (confirmado no push `39a2805`). Custo: builds cancelados pelo `ignoreCommand` contam na cota de deployments.
- Identidade do participante: nome + cookie anônimo, sem login.
- Imagens dos gatos por **URL** (sem upload).
- Postgres hospedado (Neon ou Vercel Postgres) com pooling do Prisma; Docker Compose só para dev local e CI.
- Modelo: `Session`, `Cat`, `Participant`, `Vote` (único por participante + gato).
- E2E no Playwright com dois navegadores (apresentador e aluno).

## Visual

- Bobinho e divertido, estilo "macarrão": fontes arredondadas, gordinhas e curvas, bem de brincadeira.
- **Nada de fonte monoespaçada** nem visual sério/corporativo.
- Fontes ainda não escolhidas (candidatas: Fredoka, Baloo 2, Chewy, Bubblegum Sans, via Google Fonts).
- Cores vibrantes e animações leves (o gatinho correndo pela tela pode ser só efeito visual, com emoji ou uma sprite sheet única).

## Stack

- Frontend: Next.js + shadcn/ui
- Backend: NestJS + Prisma
- Banco: PostgreSQL
- Infra local: Docker Compose
- E2E: Playwright
- Deploy: Vercel
- CI/CD: Jenkins

## Branches

Definido em 2026-10-06. A branch principal é **`master`** (não `main`) e existe a branch de integração **`dev`**.

- Fluxo: `feature/*` (ou `docs/*`, `fix/*`) → PR → `dev` → PR → `master`. `master` é o que vai para produção na Vercel.
- Nada de push direto em `master` nem em `dev`: as duas são protegidas no GitHub (vale também para o admin).
- Check obrigatório nas duas: **`Vercel`** (passa como `success` mesmo quando o `ignoreCommand` cancela o build). Em `master` a branch também precisa estar atualizada antes do merge.
- Sem exigência de aprovação de review (repo individual). Subir para 1 aprovação se mais gente entrar.
- Quando o Jenkins existir, adicionar o contexto dele (ex.: `continuous-integration/jenkins/branch`) aos checks obrigatórios das duas branches. Multibranch Pipeline cobre `dev`, `master` e PRs.
- Production Branch da Vercel (Settings → Git) deve ser `master`.

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
