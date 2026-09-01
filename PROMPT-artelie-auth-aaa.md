# PROMPT — Reengenharia AAA de Auth, Identidade e Permissões (Arteliê-open)

Quero que você faça a reengenharia completa do sistema de autenticação, identidade
e permissões do Arteliê-open (repo: /home/kadson/artelie-open — Next.js App Router +
Better Auth + Drizzle/SQLite/Turso), elevando-o ao padrão dos SaaS open-source mais
bem avaliados do GitHub: ixartz/SaaS-Boilerplate (7.4k★),
mickasmt/next-saas-stripe-starter (3k★) e dub.co. O resultado deve ser perfeito em
cada detalhe: design, microcopy pt-BR, validações inline, estados
vazio/loading/erro/sucesso, e-mails transacionais, acessibilidade e propagação
correta de usuarioId em todas as tabelas.

REGRAS DE NEGÓCIO IMUTÁVEIS:
1. Todo usuário tem acesso TOTAL e irrestrito aos dois módulos — Ateliê E Salão.
   Não existe escolha de nicho no cadastro, não existe restrição por tipo de
   conta. "Ateliê/Salão" é apenas preferência de visualização (último módulo
   visitado), nunca filtro de dados ou permissão.
2. PARIDADE MOBILE + DESKTOP: absolutamente tudo — telas, fluxos, menus,
   validações, estados, feedbacks — deve funcionar e estar visualmente perfeito
   nas duas versões. Mobile não é afterthought: é primeiro-class. Nenhuma feature
   pode existir só no desktop nem quebrar/sumir no mobile.

═══════════════════════════════════════════════════════════
O QUE SERÁ CONSTRUÍDO
═══════════════════════════════════════════════════════════

1. AUTENTICAÇÃO COMPLETA (telas /login, /cadastro, /esqueci-senha,
   /redefinir-senha, /verificar-email):
   - Cadastro com: nome, e-mail, senha + confirmar senha, medidor de força de
     senha, aceite de termos — SEM pergunta de nicho
   - Login com "lembre de mim", link "esqueci a senha", erro inline por campo,
     mensagem clara quando e-mail não verificado + botão "reenviar verificação"
     funcional
   - Bloqueio de login quando emailVerified=false, com tela intermediária digna
   - /verificar-email: estados reais (verificando / sucesso com CTA "Ir para o
     app" / link expirado com reenvio) — NUNCA redirect cego de 1,5s
   - /redefinir-senha: validação de token server-side, confirmar senha,
     requisitos visíveis, auto-login após reset com toast
   - E-mails transacionais com template HTML branded (logo, botão CTA, fallback
     texto), responsivos (abrem perfeitos em Gmail mobile e desktop), microcopy
     pt-BR
   - TODAS essas telas: layout mobile-first, inputs com type/keyboard corretos
     (email → teclado de e-mail), botões com altura touch ≥44px, sem zoom de
     input no iOS (font-size ≥16px), sem scroll horizontal em 360px

2. IDENTIDADE & PERFIL:
   - UserNav com avatar (iniciais), nome e menu (Perfil, Alterar senha, Sair):
     dropdown no desktop, item de menu na navegação mobile (drawer/bottom nav
     conforme padrão já existente no app) — sempre visível, sempre identificável
   - /usuario/perfil reestruturado em abas (desktop) / seções colapsáveis ou
     sub-rotas (mobile): Dados pessoais | Dados da empresa | Preferências
     (inclui "módulo inicial ao entrar": Ateliê/Salão — preferência de UX apenas)
   - Link no menu principal (hoje a página está órfã), breadcrumb, toast de
     sucesso; formulários com um campo por linha no mobile (nunca grid apertado)

3. PERMISSÕES & INTEGRIDADE DE DADOS (o bug raiz):
   - Auditoria do schema: toda tabela de domínio (item, cliente, pedido, insumo,
     financeiro, producao, itemFoto, movimentacaoEstoque) DEVE ter usuario_id
     NOT NULL + FK + índice; migration segura com backfill
   - TODA query e action filtra por usuarioId do usuário logado — e SOMENTE por
     usuarioId: nenhuma query de domínio filtra por tipo/nicho
   - Eliminar a dualidade user.tipo vs cookie artelie-tipo como FONTE DE DADOS:
     vira exclusivamente preferência de UI. Qualquer filtro por tipo residual
     que "esconda" itens criados deve ser removido
   - Rotas de API (/api/itens/[id]/fotos etc.) com ownership check → 404/403
   - Guard server-side em todas as rotas (app): redirect /login sem sessão, sem
     flash de conteúdo

═══════════════════════════════════════════════════════════
COMO SERÁ CONSTRUÍDO
═══════════════════════════════════════════════════════════

Distribua sub-agentes, cada um dono de um item individualmente:

- SUB-AGENTE 1 — Auth Backend: Better Auth config, e-mails, verificação,
  bloqueio de login, reset, migration usuario_id
- SUB-AGENTE 2 — Telas de Auth (UX/UI): as 5 páginas com validação inline,
  estados, microcopy pt-BR, a11y — obrigatoriamente mobile-first
- SUB-AGENTE 3 — Identidade: UserNav (desktop + mobile), perfil em
  abas/seções, menu, breadcrumbs
- SUB-AGENTE 4 — Permissões & Dados: auditoria de TODAS as queries/actions/rotas
  API, remoção de filtros por nicho, ownership checks, teste de isolamento entre
  2 usuários (A nunca vê dados de B, nem por ID direto) e teste de visibilidade
  total (mesmo usuário vê dados Ateliê + Salão sempre)
- SUB-AGENTE 5 — Testes: unit (vitest) + E2E (playwright) rodando em DOIS
  viewports (1280px desktop + 375px mobile) no projeto Playwright: signup →
  verify → login → criar item em Ateliê → vê na listagem → alterna para Salão →
  cria serviço → logout → usuário B não acessa nada de A → reset de senha
  completo — tudo nos dois viewports

Faça /loop em cada item e mantenha um sub-agente SEPARADO como CRÍTICO RIGOROSO
que valida visualmente (screenshots Playwright em 1280px E 375px, side a side) e
funcionalmente cada entrega contra os 3 apps de referência (clonar e inspecionar
o auth flow de ixartz/SaaS-Boilerplate e mickasmt/next-saas-stripe-starter como
ground truth). O crítico deve comparar CADA tela nos dois viewports e REJEITAR
com lista objetiva de gaps se uma delas não estiver no nível — o responsável
corrige e reenvia.

═══════════════════════════════════════════════════════════
QUANDO PARAR / ENTREGAR
═══════════════════════════════════════════════════════════

Não pare até que o sub-agente crítico aprove CADA fluxo, em DESKTOP e em MOBILE,
em comparação cega com o fluxo equivalente do app de referência, respondendo:
"qual dos dois eu usaria?" — se a resposta não for o Arteliê, continua o loop.

Critérios objetivos de aceite (todos verificáveis):
- [ ] E2E verde nos viewports 1280px e 375px: signup→verify→login→CRUD nos dois
      módulos→logout→reset
- [ ] Screenshots side-by-side (desktop + mobile) de cada tela de auth aprovados
      pelo crítico contra os apps de referência
- [ ] Zero scroll horizontal e zero elemento tocável <44px em 360-375px
- [ ] E-mails de verificação/reset renderizam corretamente em cliente mobile
- [ ] Usuário logado identificável em ≤1 olhada em qualquer tela, nos dois
      formatos
- [ ] Zero query sem filtro usuarioId (auditável por grep)
- [ ] Zero query de domínio filtrando por tipo/nicho (auditável por grep)
- [ ] Mesmo usuário cria e lista dados em Ateliê E Salão sem trocar nada
- [ ] Nenhuma mensagem de erro em inglês exposta ao usuário
- [ ] Zero console errors / hydration warnings nas telas de auth
- [ ] Lighthouse a11y ≥ 95 nas 5 páginas de auth (mobile e desktop)
- [ ] Teste 2-usuários prova isolamento total de dados

Distribua sub-agentes e ultracode.
