# 00 — Inventário de produtos e plano de auditoria

- Data: 2026-11-20
- Ambiente: somente leitura (conector Notion). Nenhuma escrita no workspace.
- Status da etapa: **parcialmente concluído** (triagem de topo; sem inventário de schema nesta etapa)

## 1. Verificação do ambiente

| Item | Status | Observação |
|---|---|---|
| CLAUDE.md do kit | Lido integralmente, aplicado manualmente | Está só no ZIP; não está no repositório, então o Claude Code não o carrega sozinho |
| 7 Skills do kit | Lidas integralmente, aplicadas manualmente | `notion-expert`, `notion-formula-review`, `ux-ui-auditor`, `product-design`, `qa-testing`, `conversion-copywriting`, `digital-product-audit`. Não aparecem na lista de Skills da sessão porque não estão em `.claude/skills/` do repositório |
| Templates de relatório (`audits/templates/`) | Lidos | Serão usados como base |
| Conexão Notion | OK | Conector oficial do Notion (MCP). Workspace "Espaço de Natália Rego" |
| Escopo de leitura | Páginas privadas: 9 itens de topo. Páginas compartilhadas: 0 |

## 2. Inventário do workspace (páginas de topo)

| # | Página | Classificação | É produto? | Última edição | Evidência |
|---|---|---|---|---|---|
| E1 | Planejamento de Aulas | **Educação/Professores** | Sim (provável) | 07/10/2026 | Subtítulo "feito por professora, para professor"; views Hoje, Próximas Avaliações, Pendências da Semana, Planejamento Semanal; menu `Menu.pjt01` com Turmas e Registro de Aulas |
| F1 | Dashboard Financeiro [ATUALIZADO] | **Finanças Pessoais** | Sim, versão principal | 15/11/2026 | 12 bases; Guia Inicial com regras de terceiros/reembolso; views A receber / A pagar |
| F2 | Dashboard Financeiro [2.0] | **Finanças Pessoais** | Variante | 12/11/2026 | Mesma estrutura de F1, sem os blocos A receber/A pagar nem as regras de terceiros |
| F3 | Dashboard Financeiro 2026/2027 | **Finanças Pessoais** | Variante | 12/11/2026 | Como F2, mas com o passo `◀ Mês anterior` no guia de novo ano, que F1 e F2 não têm |
| F4 | [ANTIGO]DashBoard Financeiro 2027 | **Finanças Pessoais** | Versão descontinuada | 12/10/2026 | Já exportada em `export/` e auditada em `AUDITORIA.md` |
| F5 | Controle Financeiro 2026 | **Finanças Pessoais** | A confirmar | 06/10/2026 | Geração anterior (Detalhamento/Resumo Mensal, 8 botões). Pode ser seu uso pessoal com dados reais |
| — | CHECKLIST | Material de apoio de F1 | Não | 26/10/2026 | Checklist de lançamento + "Material de venda — Dashboard Financeiro (Hotmart)" + "Guia de implantação — meses automáticos" |
| — | Bem vinda ao Caos! | Fora de escopo (pessoal) | Não | 26/10/2026 | Painel pessoal: Vida, Pessoal, Tarefas, atalhos |
| — | Doutorado | Fora de escopo (pessoal) | Não | 26/10/2026 | Artigos, matérias, cronograma |

Observações da triagem (fatos, não achados):
- F1, F2 e F3 têm databases próprias (IDs diferentes): são cópias que divergiram, não views da mesma base.
- E1 parece estar em uso real: turmas como "M - 2001", "SM - 3ª Série" e aulas como "Aula 21 - Lei da velocidade". Por isso conta como **produção com dados de estudantes**.
- F1 tem registros "Untitled" e "Teste - 3x" em Registro de Despesas (14 a 16/11), possivelmente restos de teste.

## 3. Limitações de inspeção (todos os templates)

| Limitação | Efeito na auditoria |
|---|---|
| O conector devolve só a referência das fórmulas (`formulaResult://…`), sem o código | `notion-formula-review` fica limitada a validar resultados indiretos (filtros, views, recálculo manual) até você colar o código das fórmulas |
| Automações não são expostas | Gatilhos (`Mês (auto)`, parcelas) só verificáveis por teste em cópia ou print |
| Botões aparecem como blocos `unknown` | Rótulos e ações dos botões não são legíveis |
| Sem renderização visual nem mobile | UX/UI baseada na estrutura dos blocos. Cores, capa, gráficos e celular dependem de prints ou vídeo |
| Limite de consultas SQL do Notion | Bases grandes podem ficar com amostragem parcial |
| Testes funcionais exigem escrita | QA só em cópia sandbox. Duplicar no Notion exige sua aprovação específica, ou você duplica |

## 4. Plano de auditoria por template

### E1 — Planejamento de Aulas (Educação)
- **Skills:** todas as 7. `notion-expert` (inventário completo, nunca feito), `notion-formula-review` (casos docentes), `ux-ui-auditor`, `product-design`, `qa-testing`, `conversion-copywriting`, `digital-product-audit`.
- **Escopo:** inventário de Menu.pjt01, Turmas, Registro de Aulas, Avaliações, Pendências e Planejamento Semanal; relações turma↔disciplina↔aula; filtros "Hoje"/"Semana"; reutilização entre turmas; carga de preenchimento; onboarding.
- **Falta coletar:** se é produto à venda; público (Fundamental II, Médio, outro); se existe material de venda; se há versão limpa para comprador; código das fórmulas; prints desktop/mobile.
- **Restrições:** tratar como produção. Nenhum nome de turma, aluno ou conteúdo real nos relatórios. QA só em cópia com dados fictícios.

### F1 — Dashboard Financeiro [ATUALIZADO] (Finanças) — principal
- **Skills:** todas as 7, com foco em `notion-formula-review` e `qa-testing`.
- **Escopo:** atualizar a `AUDITORIA-v2.md` (feita hoje) com as mudanças de 15/11 (Responsáveis/Terceiros, A receber/A pagar, Devo para, Balanço); fechar os itens pendentes do CHECKLIST; revisar o "Material de venda (Hotmart)" contra o produto real (`conversion-copywriting` + `digital-product-audit`).
- **Falta coletar:** código das fórmulas críticas; prints ou vídeo dos 8 botões e do celular; confirmação do plano Notion exigido (Plus?); se os registros "Untitled"/"Teste - 3x" são intencionais.
- **Restrições:** os casos financeiros mínimos da Skill (parcelado, estorno, virada de mês/ano, mês vazio, centavos) só viram PASS com execução em cópia.

### F2 [2.0] e F3 [2026/2027] — variantes
- **Skills:** só `notion-expert` (comparação de schema) + `product-design` (decisão de versão).
- **Escopo:** diff estrutural contra F1 para descobrir o que cada uma tem de único (ex.: `◀ Mês anterior` em F3) e recomendar qual versão vira o produto.
- **Falta coletar:** sua intenção para cada uma (venda, backup, experimento).

### F4 [ANTIGO] — descontinuada
- **Skills:** nenhuma nova. Já coberta por `AUDITORIA.md` + `export/`.
- **Proposta:** não auditar de novo; só registrar como arquivada.

### F5 — Controle Financeiro 2026
- **Skills:** depende da sua resposta. Se for produto: `notion-expert` + `digital-product-audit` (triagem). Se for uso pessoal: fora de escopo.
- **Restrições:** pode conter dados financeiros reais. Nenhum valor será reproduzido em relatório.

## 5. Próximo passo
Aguardando aprovação: ordem das auditorias, respostas da seção 4 e se a cópia sandbox será criada por você ou por mim (com aprovação específica).
