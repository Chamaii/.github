<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/banner-dark.svg">
  <img alt="ChamaAi — marketplace de serviços de mão de obra autônoma no Brasil" src="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/banner-light.svg" width="100%">
</picture>

<br><br>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/selos-dark.svg">
  <img alt="Status: em desenvolvimento · Versão 0.8.0 · Node ≥ 22 · Licença proprietária" src="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/selos-light.svg" width="615">
</picture>

</div>

<br>

## O que a gente está construindo

Contratar um pedreiro, um eletricista ou uma diarista no Brasil ainda é um exercício de fé. Você pede indicação no grupo do prédio, recebe um número de telefone, combina tudo por áudio no WhatsApp, paga no Pix e torce para dar certo. Não existe orçamento formal, não existe garantia, não existe para onde recorrer quando o serviço sai errado — e do outro lado, o profissional também não tem como provar que é bom, nem como receber sem depender da palavra do cliente.

A **ChamaAi** é um marketplace que coloca esse encontro dentro de um fluxo com regras claras: o cliente descreve o serviço e encontra profissionais perto dele, recebe um orçamento de verdade, aprova, acompanha a execução pelo chat e só libera o pagamento quando o serviço termina. O prestador passa por verificação de identidade antes de aparecer na busca, e o dinheiro fica retido na plataforma até a conclusão. Se algo der errado, existe uma janela de contestação e um processo de disputa com desfecho registrado — não um bloqueio no WhatsApp.

<br>

## Do "preciso de alguém" ao "valeu, ficou ótimo"

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/fluxo-dark.svg">
  <img alt="Ciclo do serviço: busca, orçamento, aceite, execução, pagamento e avaliação" src="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/fluxo-light.svg" width="100%">
</picture>
</div>

Cada etapa desse ciclo é uma transição explícita de uma máquina de estados que vive em `packages/domain`, isolada de framework, de banco e de HTTP. Não dá para pular do orçamento direto para o pagamento, não dá para avaliar um serviço que nunca foi executado, e cancelamento e reagendamento têm regra e penalidade definidas em código — não em `if` espalhado pelo controller. Em volta do caminho feliz existem chat em tempo real com heurística de detecção de conteúdo, moderação com congelamento e contestação, trilha de auditoria e disputa com tratamento de reincidência.

<br>

## Como isso está montado

Um monorepo só, com três aplicações consumindo o mesmo contrato tipado de ponta a ponta — o cliente tRPC do web e do mobile falam com a API sem nenhum schema duplicado no meio.

| Onde | O que tem dentro |
| :-- | :-- |
| **`apps/api`** | NestJS + tRPC, Drizzle ORM sobre Postgres/PostGIS, BullMQ para jobs e Socket.IO para o chat |
| **`apps/web`** | Next.js 16 (App Router) com React 19 e Tailwind CSS 4 |
| **`apps/mobile`** | Expo (React Native), mesmo cliente tRPC do web |
| **`packages/domain`** | Regras de negócio puras — máquina de estados, precificação, disputas, penalidades |
| **`packages/contracts`** | Schemas Zod compartilhados entre API, web e mobile |
| **`packages/ui`** | Tokens visuais e componentes que web e mobile dividem |

<br>

## Stack

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/stack-dark.svg">
  <img alt="Aplicações: TypeScript, NestJS, tRPC, Next.js, React, Expo, Tailwind CSS. Dados e tempo real: PostgreSQL, PostGIS, Drizzle ORM, Redis, BullMQ, Socket.IO, Zod. Serviços: Clerk, Mercado Pago, Didit, Cloudflare R2, Sentry. Engenharia: Turborepo, pnpm, Vitest, Playwright, Docker, GitHub Actions, Fly.io, Vercel" src="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/stack-light.svg" width="927">
</picture>

<br><br>

## O que a gente leva a sério

**Acessibilidade entra antes do CSS.** Os tokens de UI não são "o que ficou bonito": o alvo de toque mínimo é de 48px porque o público inclui gente de luva, no sol, com o celular na mão suja de tinta; o corpo de texto é 17px para não disparar zoom em mobile web; o texto principal fica em torno de 19:1 de contraste e o secundário em 11:1, os dois acima de AA; e nenhum elemento interativo da base zera o `outline` sem colocar outro no lugar. Cada decisão dessas está comentada junto do valor, em `packages/ui/src/tokens`.

**Privacidade é um job de CI, não uma promessa.** Todo push roda uma auditoria automática de PII em logs e em respostas da API — se um CPF, um telefone ou um endereço vazar para onde não devia, o pipeline quebra antes de virar produção. A documentação de LGPD e o canal do DPO ficam versionados junto do código, em `docs/lgpd`.

**Domínio não conhece framework.** As regras de negócio não importam NestJS, não importam Drizzle e não sabem o que é uma requisição HTTP. Isso não é preferência estética: existe um job de `lint:boundaries` no CI que reprova o PR se alguém cruzar essa fronteira. O resultado é que a lógica que decide se um cancelamento gera penalidade roda em teste unitário puro, em milissegundos, sem subir banco.

<br>

## Como a gente trabalha

`main` é produção e `dev` é integração — ninguém commita direto em nenhuma das duas. Todo trabalho sai em branch (`feat/`, `fix/`, `chore/`) e volta em Pull Request contra `dev`, com commits em [Conventional Commits](https://www.conventionalcommits.org/) e versionamento [SemVer](https://semver.org/) registrado em commit próprio.

Antes de qualquer merge, o CI roda cinco portões: lint com fronteira de domínio, testes unitários e de integração com Postgres/PostGIS e Redis de serviço, auditoria de PII, E2E em Playwright e build. O deploy é manual e explícito — API na Fly.io (região GRU) e web na Vercel — porque subir produção deve ser uma decisão, não um efeito colateral de merge.

<br>

## Repositórios

| Repositório | O que é | |
| :-- | :-- | :-- |
| [**Chamai**](https://github.com/Resnow-Team/Chamai) | Monorepo do produto — API, web, mobile e pacotes compartilhados | 🔒 privado |

<!-- Equipe: adicione aqui os perfis do time, ex.:
[<img src="https://github.com/usuario.png" width="56" alt="Nome"/>](https://github.com/usuario)
-->

<br>

---

<div align="center">
<br>
<img src="https://raw.githubusercontent.com/Resnow-Team/.github/main/assets/logo.svg" width="26" alt="">
<br><br>
<sub><b>Resnow Team</b> · Rio de Janeiro, Brasil<br>
Código proprietário — todos os direitos reservados.</sub>
<br><br>
</div>
