# Relatório de auditoria — Dashboard Financeiro [ATUALIZADO] (F1)

- Data: 2026-11-20
- Versão: página "Dashboard Financeiro [ATUALIZADO]" (última edição da home: 15/11/2026; Registro de Despesas: 16/11/2026)
- Segmento: Finanças pessoais
- Origem: workspace Notion, via conector oficial (MCP)
- Ambiente: leitura em produção (nenhuma escrita no original) + **cópia sandbox** autorizada em 20/11/2026: "🧪 SANDBOX QA — Dashboard Financeiro (não usar)" (`41e4e98d4af3816087ccd72f2d473292`)
- Status: **Concluído parcialmente** (técnico, UX estrutural e comercial cobertos; fórmulas parciais; QA funcional iniciado na sandbox)
- Base: este relatório revalida e reorganiza a `AUDITORIA-v2.md` no formato do kit e acrescenta a auditoria do material de venda (Hotmart), que não tinha sido feita

---

## 1. Resumo executivo

**Parecer: Não pronto para lançamento.**

O núcleo técnico é bom e diferenciado. A competência do cartão pelo vencimento da fatura, a separação entre consumo e investimento e a coluna de pendências estão bem resolvidas na estrutura. O que impede a venda hoje é um problema de produto, não de engenharia:

1. **O material de venda promete "funciona no plano gratuito"**, mas o próprio CHECKLIST tem em aberto "Avisar na página de venda que precisa do Notion Plus". O mês de cada lançamento depende de uma automação de banco de dados (COM-001).
2. **Promessas que a versão atual não cumpre**: "as parcelas aparecem sozinhas" (elas aparecem, mas sem categoria) e "mês de exemplo para ver tudo funcionando" (o dashboard mostra o mês atual, não o exemplo) (COM-002, TEC-002, UX-001).
3. **Dados de teste no template** que entram no mês do comprador (TEC-003).
4. **`Gastos de consumo` inclui gastos de terceiros** (FORM-001), ao contrário do que o Guia promete. Isso aparece na revisão das fórmulas enviadas por print em 20/11 (seção 5.1).

Os três primeiros lotes do plano de correção (seção 7) resolvem os bloqueios sem mexer na arquitetura.

## 2. Escopo e método

| Skill | Uso nesta auditoria |
|---|---|
| `notion-expert` | Schema de Registro de Despesas, 📅 Meses e 🗂️ Grupos; 21 views de Despesas; layout das páginas |
| `notion-formula-review` | Só verificação indireta: recálculo manual a partir dos dados brutos. O código não é acessível (seção 3) |
| `ux-ui-auditor` | Walkthrough heurístico pela estrutura dos blocos (sem tela renderizada) |
| `product-design` | Jornada compra → duplicação → configuração → primeiro lançamento |
| `qa-testing` | Matriz montada; **nenhum caso executado** (sem cópia sandbox autorizada) |
| `conversion-copywriting` | Revisão linha a linha do "Material de venda — Dashboard Financeiro (Hotmart)" |
| `digital-product-audit` | Teste de prontidão e parecer |

Evidências consultadas: home do F1; Configurações Iniciais; schema e todos os 22 registros de Registro de Despesas (por view salva); schema de 📅 Meses; base e views de 🗂️ Grupos; página CHECKLIST; Material de venda (Hotmart); `AUDITORIA-v2.md`.

## 3. Cobertura e limitações

| Dimensão | Cobertura | Score 0–5 | Evidência |
|---|---|---:|---|
| Integridade técnica | Parcial (3 de 12 bases revalidadas hoje; as outras vêm da v2) | 3 | Schema coerente e documentado. Dependência de automação, parcelas sem categoria e lixo de teste |
| Fórmulas e cálculos | Parcial (6 fórmulas lidas por print) | 3 | Lógica do cartão correta e robusta. `Gastos de consumo` soma terceiros (FORM-001); `⚠️ Pendência` deixa lacunas (FORM-002) |
| Arquitetura da informação | Parcial | 3 | Módulos claros. Orçamento frágil (432 registros, select por nome) |
| Usabilidade e navegação | Parcial (estrutural) | 2 | Configuração oculta, compra no cartão "some", 21 views, Tipo de Pagamento ambíguo |
| Visual e consistência | Sem acesso visual | não avaliado | Só estrutura dos blocos; ver `AUDITORIA-v2.md` §4 como hipóteses |
| Acessibilidade | Sem acesso visual | não avaliado | — |
| Onboarding | Parcial | 3 | Guia de 8 passos com "Teste você mesmo" é muito bom. Fica oculto, e o callout da dívida está desatualizado |
| Adequação ao público | Parcial | 3 | Bom para quem usa cartão. Excesso de regras de casal e terceiros para quem é sozinho |
| Prontidão comercial | Boa | 1 | Contradição sobre plano gratuito, promessas não sustentadas, placeholders abertos |

**Score global: 2,6** (média de 7 dimensões avaliadas de 9; todas com cobertura parcial). O score não substitui os achados críticos COM-001 e TEC-001.

Limitações que não são defeito do template:

| Limitação | Efeito |
|---|---|
| `formulaCode://` não é suportado pelo conector ("URL type formulaCode not currently supported") | Nenhuma fórmula foi lida. Toda conclusão sobre fórmula é inferência pelo comportamento das views ou pelos dados |
| Automações e botões não são expostos (blocos `unknown`) | Rótulos e ações dos 7 botões do ⚡ Lançamento rápido e dos botões de propriedade não verificados |
| Cota de consultas SQL/rows do workspace esgotada durante a auditoria | Leitura por views salvas. Não consultei Entradas, Orçamento, Cartões, Dívidas, Metas, Subcategorias, Responsáveis e Base nesta rodada |
| Sem tela renderizada | Visual, gráficos e celular não avaliados |
| Sem cópia sandbox | Nenhum teste funcional executado |

## 4. Inventário técnico (revalidado hoje)

**Registro de Despesas** — 40 propriedades: 11 de entrada, 25 fórmulas, 3 rollups e 1 botão. Todas as auxiliares têm descrição ("Auxiliar… Não editar"), o que é um ponto forte.

- Entradas do usuário: Descrição, Data da Despesa (DD/MM/AAAA), Valor (R$), Tipo de Pagamento (7 opções), Natureza (Fixa/Variável), Categoria (relação, 1), 👤 Responsável, Cartão de Crédito (relação, 1), Parcelas (select 2–12), Dívida, Meta, Devo para, Pago.
- Preenchida por automação: `Mês (auto)`. A descrição diz: "Preenchido automaticamente pela automação a partir da data".
- Template de página padrão: "Nova despesa".
- Views: 21 no total (📋 Todas, Jan…Dez, 🔎 Auditoria de meses, 💳 Cartão, 💸 Pix, 🏦 Dívidas, 📈 Investimentos, 📌 Fixas x Variáveis, 🧾 Faturas por mês, ✅ Faturas a pagar). As views Jan…Dez filtram por `Chave` terminando em "-MM", então juntam todos os anos.

**📅 Meses** — totais por rollup de `Mês (auto)` (Cartões, Pagos à vista, Dívidas, Investimentos, Terceiros, Total Entradas, Reembolsos), fórmulas derivadas (Gastos de consumo, Sobra, Taxa de poupança, Saldo em caixa, vs mês anterior) e fórmulas paralelas "(pela data)" mais uma `Conferência`.

**🗂️ Grupos de Categoria** — Padrão R$ / Padrão %, Gasto no mês, Gasto no ano, Média por mês. Duas views: tabela "📊 Resumo por grupo" e **gráfico de rosca** "📊 Gráfico do mês". Há também um gráfico "Resultado mês a mês" (citado na descrição de `É ano atual` em Meses).

**Registros atuais de Despesas (22)**:

| Data | Descrição | Valor | Tipo | Observação |
|---|---|---:|---|---|
| — | Nova despesa | — | — | Linha vazia |
| 10/09/2026 | Exemplo — Streaming | 55 | Cartão | |
| 05/10/2026 | Exemplo — Aluguel | 1.500 | Pix | |
| 05/10/2026 | Exemplo — Farmácia | 120 | Cartão | |
| 06/10/2026 | Exemplo — Aporte reserva | 500 | Investimento | Sem Responsável |
| 09/10/2026 | Exemplo — Jantar fora | 320 | Pix | |
| 10/10/2026 | Teste - 3x (01/03) | 100 | Cartão | Com categoria |
| 12/10/2026 | Exemplo — Combustível | 350 | Pix | |
| 20/10/2026 | Exemplo — Delivery do mês | 460 | Pix | |
| 10/11/2026 | Teste - 3x (02/03) | 100 | Cartão | **Sem categoria** |
| 10/11/2026 | (sem descrição) | 100 | Cartão | **Sem cartão** |
| 10/12/2026 | Teste - 3x (03/03) | 100 | Cartão | **Sem categoria** |
| 14/10/2026 a 14/07/2027 | Exemplo — Empréstimo 1/10 … 10/10 | 500 cada | Dívida | Sem Responsável; nenhuma marcada como Pago |

## 5. Achados

Severidade: Crítica / Alta / Média / Baixa. Esforço: P / M / G. A referência "v2 P#" aponta para o item equivalente na `AUDITORIA-v2.md`.

### COM-001 — O material de venda promete plano gratuito sem comprovação
- **Dimensão:** Prontidão comercial. **Local:** Material de venda (Hotmart): Requisitos, FAQ "Preciso pagar o Notion?" e roteiro de vídeo.
- **Evidência:** a página de vendas diz "funciona no plano gratuito" e o FAQ responde "Não. O painel funciona no plano gratuito". Ao mesmo tempo:
  - O CHECKLIST tem em aberto "Avisar na página de venda que precisa do Notion Plus".
  - O item 7 do próprio material ainda manda "Confirmar que as automações funcionam na cópia".
  - `Mês (auto)`, a base de todos os rollups de Meses, é preenchido "pela automação".
  - O template tem pelo menos 2 views de gráfico.
- **Hipótese a confirmar (confiança média):** pelo que conheço das regras do Notion, automações de banco de dados com ação de editar propriedade e mais de um gráfico exigem plano pago. Confirme na documentação atual e numa conta Free nova.
- **Consequência:** se a hipótese se confirmar, o comprador Free vê o dashboard zerado. Isso gera pedido de reembolso e reclamação por propaganda enganosa.
- **Severidade:** Crítica. **Confiança:** Alta para a contradição; Média para o efeito no Free.
- **Recomendação:** não publicar até o teste em conta Free. Resultado A: se funcionar, mantenha o texto e anexe o vídeo do teste como evidência interna. Resultado B: se não funcionar, troque por "Requer Notion Plus (ou superior) para o preenchimento automático dos meses" ou faça o redesenho de TEC-001.
- **Esforço:** P (texto) ou G (redesenho). **Aceite:** requisito de plano escrito igual na página de vendas, no FAQ, no vídeo e na mensagem de entrega, com base no teste documentado.

### COM-002 — Promessas absolutas ou não sustentadas pela versão atual
- **Dimensão:** Prontidão comercial / copy. **Local:** Material de venda, §1 e §2.

| Texto atual | Problema | Proposta |
|---|---|---|
| "Controle Total do Seu Dinheiro" (nome) | Promessa absoluta | "Dashboard Financeiro no Notion — cartões, parcelas e orçamento em um só lugar" |
| "Saiba **exatamente** para onde vai o seu dinheiro" | Depende do usuário lançar tudo | "Veja para onde vai o seu dinheiro, mês a mês" |
| "organiza sua vida financeira **sozinho**" | Lançar é manual; o próprio texto diz isso em "Para quem NÃO é" | "Você lança o gasto e o painel organiza por mês, categoria e cartão" |
| "lance uma compra em até 12x e as parcelas aparecem sozinhas" | Hoje as parcelas 2…N nascem sem Categoria (TEC-002) | Manter só depois de corrigir TEC-002 e testar 3x e 12x |
| "Mês de exemplo preenchido, para você ver tudo funcionando" | O dashboard mostra o mês atual; o exemplo de out/2026 não aparece na home (UX-001) | Corrigir UX-001 ou trocar por "Vídeo e guia mostrando um mês completo" |
| "saldo em caixa" | O campo é resultado do mês, não saldo de conta (UX-004) | "resultado do mês" |
| "Construir hábitos financeiros saudáveis" (Guia) | Benefício comportamental não verificável | Pode ficar como aspiração, sem "garante" |

- **Consequência:** expectativa maior que o produto, que vira reembolso dentro dos 7 dias.
- **Severidade:** Alta. **Confiança:** Alta. **Esforço:** P.
- **Aceite:** toda afirmação do material corresponde a um caso PASS na matriz de testes ou é reescrita.

### COM-003 — Pendências de lançamento no material
- **Evidência:**
  - Placeholders abertos: `[LINK DE DUPLICAÇÃO]`, `[SEU E-MAIL OU WHATSAPP DE SUPORTE]` e `[Decidir: compradores recebem a versão 2…]`.
  - O roteiro de prints e de vídeo usa out/2026; o próprio checklist pede para atualizar o mês de exemplo.
  - "Meses de 2026 a 2028" num produto vendido para 2027.
- **Severidade:** Média. **Confiança:** Alta. **Esforço:** P.
- **Aceite:** nenhum colchete no material; prints e vídeo gravados a partir da cópia MASTER final.

### TEC-001 — Cálculo mensal depende de automação (v2 P1)
- **Local:** `Registro de Despesas.Mês (auto)` → rollups de 📅 Meses.
- **Evidência:** a descrição da propriedade cita a automação. Em Meses, Cartões, Pagos à vista, Dívidas, Investimentos e Total Entradas são rollups dessa relação. Já existem fórmulas paralelas "(pela data)" e uma `Conferência`.
- **Consequência:** se a automação não rodar (plano, cópia ou falha), os totais do mês ficam zerados sem nenhum aviso.
- **Severidade:** Crítica (se COM-001 se confirmar) ou Média (se o Free funcionar). **Confiança:** Média.
- **Recomendação:** depende do teste Free. Opção 1: KPIs a partir de "(pela data)". Opção 2: requisito Plus declarado. Em qualquer caso, mostrar um alerta quando `Conferência` divergir.
- **Esforço:** G. **Risco de regressão:** alto (todas as views de Meses, o Orçamento e o Resumo anual).
- **Aceite:** com a automação desligada numa cópia, os totais de out/2026 continuam R$ 5.800 / R$ 3.305 / R$ 2.495.

### TEC-002 — Parcelas geradas perdem a Categoria (v2 P2) — revalidado hoje
- **Evidência:** "Teste - 3x" 01/03 tem Categoria; 02/03 (10/11/2026) e 03/03 (10/12/2026) estão sem ela.
- **Consequência:** as parcelas futuras saem do orçamento por grupo, do gráfico e do resumo anual.
- **Severidade:** Alta. **Confiança:** Alta (no dado). A causa provável é a ação da automação ou do botão, que não foi inspecionável.
- **Recomendação:** a ação que cria as parcelas deve copiar Categoria, Natureza e 👤 Responsável.
- **Esforço:** P. **Aceite:** QA-003 e QA-004 em PASS.

### TEC-003 — Dados de teste no template (v2 P4) — revalidado hoje
- **Evidência:** linha vazia "Nova despesa"; despesa sem descrição de R$ 100 em 10/11/2026, tipo Cartão, sem cartão; 3 parcelas de "Teste - 3x".
- **Consequência:** em nov e dez/2026, o comprador já vê gastos que não são dele.
- **Severidade:** Alta. **Confiança:** Alta. **Esforço:** P.
- **Aceite:** busca por "Teste", título vazio e "Nova despesa" sem resultado na MASTER; lixeira vazia.

### TEC-004 — Lançamentos sem Responsável nos exemplos (novo)
- **Evidência:** as 10 parcelas de "Exemplo — Empréstimo" e "Exemplo — Aporte reserva" não têm 👤 Responsável. Pelo guia: Eu R$ 985 + Casa R$ 1.820 = R$ 2.805, enquanto o consumo do mês é R$ 3.305. A parcela de R$ 500 não é de ninguém.
- **Consequência:** a soma dos responsáveis não fecha com o total, e o comprador desconfia dos números. Não sei se `⚠️ Pendência` acusa a falta de responsável (fórmula não legível).
- **Severidade:** Baixa. **Confiança:** Alta (no dado). **Esforço:** P.
- **Recomendação:** atribuir Responsável aos exemplos e fazer o botão da dívida copiar o Responsável (item já aberto no CHECKLIST).
- **Aceite:** soma por responsável = Gastos de consumo do mês.

### TEC-005 — Orçamento frágil (v2 P6)
432 registros pré-criados e `Grupo` como select de texto que precisa ter o mesmo nome do título em Grupos. Renomear um grupo quebra o orçamento em silêncio. **Severidade:** Média. **Esforço:** G. A recomendação da v2 continua válida (view de Grupos ou relação).

### TEC-006 — Opções de "Parcelas" fora de ordem (novo)
- **Evidência:** a ordem das opções no select é 10, 11, 12, 2, 3, …, 9.
- **Consequência:** atrito ao escolher; é fácil pegar 10 em vez de 2.
- **Severidade:** Baixa. **Confiança:** Alta (ordem no schema; a exibição real não foi confirmada). **Esforço:** P.
- **Aceite:** opções em ordem 2 → 12.

### UX-001 — Dashboard abre sem os exemplos (v2 P3)
Todos os blocos de "📍 Este mês" usam `É mês atual`. Os exemplos estão em out/2026, e as parcelas de exemplo de jan a jul/2027 vão aparecer como dívida real. **Severidade:** Alta. **Esforço:** M.

### UX-002 — Configurações Iniciais oculta (v2 P13) — revalidado hoje
- **Evidência:** propriedade `Visibilidade = Oculta` na página.
- **Consequência:** a única porta de entrada fica dentro de um toggle fechado do Guia.
- **Severidade:** Média. **Esforço:** P. **Aceite:** visível no Menu e no primeiro bloco da home.

### UX-003 — Compra no cartão "some" de Lançamentos (v2 P7)
**Severidade:** Média. **Esforço:** P.

### UX-004 — "Saldo em caixa" não é saldo (v2 P10)
- **Evidência:** a descrição do campo é "Tudo que entrou… menos tudo que saiu". Não há saldo inicial.
- **Severidade:** Média. **Esforço:** P (renomear).

### UX-005 — Tipo de Pagamento mistura forma e natureza (v2 P9) — revalidado hoje
As 7 opções continuam as mesmas, com "DDA" e sem Débito, Dinheiro e Boleto. **Severidade:** Média. **Esforço:** M (afeta 8+ fórmulas auxiliares).

### UX-006 — Termos internos e excesso de views (v2 P12) — revalidado hoje
A seção "Debug" continua no layout da despesa; há 21 views; "🔎 Auditoria de meses" é entregue ao comprador. **Severidade:** Baixa. **Esforço:** P.

### UX-007 — Alerta vazio sem orçamento engana (v2 P5)
**Severidade:** Média. **Esforço:** P.

### COPY-001 — Callout da página de dívida desatualizado (v2 P8)
Cita "Criar 10 parcelas", a base "Saídas" e "Status Geral", que não existem. **Severidade:** Média. **Esforço:** P.

### PD-001 — Complexidade para quem é sozinho (v2 P11)
O Guia tem 9 regras, várias de terceiros e casal, e há os blocos A receber e A pagar. **Severidade:** Média. **Esforço:** M. **Recomendação:** modo Essencial com o Avançado recolhido.

### PD-002 — Cinco cópias do dashboard no workspace (v2 P16)
Há risco de publicar a cópia errada, e F2 e F3 divergem de F1 (F3 tem `◀ Mês anterior`). **Severidade:** Média. **Esforço:** P (decisão + arquivar).

## 5.1 Revisão de fórmulas (notion-formula-review)

Fonte: 6 prints do editor de fórmulas enviados pela autora em 20/11/2026. As fórmulas abaixo foram **transcritas de imagem**; conferir a grafia antes de reaproveitar.

### Fórmulas lidas

**A. `Registro de Despesas → Mês de Pagamento (cartão)`**
```
lets(d, Data da Despesa,
  F, toNumber(format(Dia do Fechamento (cartão))),
  V, Dia do Vencimento (cartão),
  ini, dateSubtract(d, date(d) - 1, "days"),
  fech, if(empty(F) or F == 0, ini, if(date(d) >= F, dateAdd(ini, 1, "months"), ini)),
  venc, if(empty(V) or V == 0 or empty(F) or F == 0, fech,
           if(V > F, fech, dateAdd(fech, 1, "months"))),
  if(empty(d), d, venc))
```
Como funciona:
- `ini` é o 1º dia do mês da compra.
- Compra no dia do fechamento ou depois vai para o ciclo seguinte (`>=`).
- Se o vencimento é maior que o fechamento, a fatura vence no mesmo mês do fechamento; senão, no mês seguinte.
- Isso cobre os dois padrões reais (ex.: Nubank fecha 25 e vence 5; Inter fecha 9 e vence 16).

**B. `Registro de Despesas → Competência`**
```
lets(ref, if(Tipo de Pagamento == "Cartão de Crédito" and !empty(Mês de Pagamento (cartão)),
             Mês de Pagamento (cartão), Data da Despesa),
  if(empty(ref), ref, dateSubtract(ref, date(ref) - 1, "days")))
```
Cartão usa o mês do vencimento; os demais tipos usam o 1º dia do mês da data. "Cartão de outra pessoa" entra pela data da compra.

**C. `📅 Meses → Gastos de consumo`**
```
Total Despesas - Investimentos no Mês
```

**D. `📅 Meses → Taxa de poupança`**
```
if(Total Entradas > 0, format(round(Sobra do mês / Total Entradas * 100)) + "%", "–")
```

**E. `Registro de Despesas → ⚠️ Pendência`**
```
ifs(empty(Descrição) or test(Descrição, "^(Novo Gasto|Novo Gasto no Cartão|Investimento|Parcela de Dívida) *$"), "⚠️ Sem descrição",
    Tipo de Pagamento == "Cartão de Crédito" and empty(Cartão de Crédito), "⚠️ Sem cartão",
    empty(Valor), "⚠️ Sem valor",
    empty(Data da Despesa), "⚠️ Sem data",
    empty(Categoria), "⚠️ Sem categoria",
    empty(Mês (auto)), "⚠️ Sem mês",
    "")
```

### Resultados observados nos prints (dados existentes, não teste com dado novo)

| Caso | Entrada | Esperado | Observado | Status |
|---|---|---|---|---|
| Inter (F=9, V=16), compra 05/10/2026 | Farmácia | 10.OUT26 | 10.OUT26 (Mês (auto) = Mês Calculado) | PASS (observação) |
| Inter, compra 10/10/2026 (depois do fechamento) | Teste 01/03 | 11.NOV26 | 11.NOV26 | PASS (observação) |
| Inter, compra 10/11/2026 | Teste 02/03 | 12.DEZ26 | 12.DEZ26 | PASS (observação) |
| Inter, compra 10/12/2026 (virada de ano) | Teste 03/03 | 01/01/2027 | Prévia da fórmula: "1 de janeiro de 2027"; grupo 01.JAN27 | PASS (observação) |
| Nubank (F=25, V=5), compra 10/09/2026 | Streaming | 10.OUT26 | Agrupado em 10.OUT26 | PASS (observação) |
| Inter, fatura de nov não paga em 20/11 | Vencimento 16/11 | "Fatura Atrasada", fatura do mês R$ 100, próxima R$ 100 | Exatamente isso | PASS (observação) |
| Meses nov/26: consumo = cartões + à vista + dívidas | 200 + 0 + 500, sem terceiros | R$ 700 | R$ 700 | PASS (observação) |
| Meses nov/26: "vs mês anterior" | 700 contra 3.305 | ▼ 79% | "▼ 79% vs out" | PASS (observação) |
| Meses nov/26: taxa de poupança sem entradas | Entradas 0 | "–" | "–" | PASS (observação) |
| `Mês (auto)` igual a `Mês Calculado` em 4 compras do Inter | — | iguais | iguais | PASS (observação). É evidência de que a automação segue a regra do cartão neste workspace |

Esses PASS confirmam a lógica central do cartão com dados reais do template. Não substituem os testes com dado novo da seção 6.

### FORM-001 — `Gastos de consumo` inclui gastos de terceiros
- **Local:** `📅 Meses → Gastos de consumo` = `Total Despesas - Investimentos no Mês`.
- **Evidência:** `Total Despesas` é o rollup (soma) da propriedade **number** de Despesas pela relação `Mês (auto)`. A única propriedade numérica de Despesas é `Valor`, então ele soma **tudo**, terceiros incluídos. Já `Cartões no Mês`, `Pagos à vista` e `Dívidas no Mês` somam as versões **"(próprio)"**, que excluem terceiros. A descrição da própria propriedade diz "(cartão + à vista + dívidas)", e o Guia diz que gasto de terceiro "não conta como gasto seu".
- **Consequência:** quando o comprador usa Responsável do tipo Terceiro (recurso anunciado: "A receber"), o gasto da outra pessoa entra no consumo. Sobra, Taxa de poupança e "vs mês anterior" ficam piores do que a realidade, e o card deixa de bater com Cartões + À vista + Dívidas. Nos dados atuais não aparece porque não há lançamento de terceiro (nov/26 bate: 200 + 0 + 500 = 700).
- **Severidade:** Alta. **Confiança:** Alta para a lógica, porque o tipo `number` identifica `Valor` sem ambiguidade. O efeito numérico não foi testado.
- **Recomendação:** `Cartões no Mês + Pagos à vista + Dívidas no Mês`, ou `Total Despesas - Investimentos no Mês - Terceiros no Mês`. Escolher depois de confirmar onde "Cartão de outra pessoa" é somado (QA-015).
- **Esforço:** P. **Risco de regressão:** baixo. Afeta Sobra, Taxa de poupança, vs mês anterior e Saldo em caixa; Saldo em caixa deve continuar incluindo terceiros, segundo a descrição dele.
- **Aceite:** QA-015 em PASS; em out/26 o consumo continua R$ 3.305.

### FORM-002 — `⚠️ Pendência` não cobre casos que afetam os totais
- **Evidência (fórmula E):** não verifica:
  1. `Tipo de Pagamento` vazio. O lançamento entra em `Total Despesas` (logo no consumo) e não entra em Cartões, À vista nem Dívidas, então o card não fecha.
  2. `👤 Responsável` vazio, o que explica TEC-004.
  3. Tipo "Cartão de outra pessoa" sem `Devo para`: some do bloco "A pagar".
  4. Tipo "Investimento" sem `Meta`: não soma no progresso da meta.
  5. Tipo "Dívida" sem `Dívida`.
  6. Valor negativo.
- `ifs` mostra só a primeira pendência. O usuário corrige uma e aparece outra.
- O passo 7 das Configurações Iniciais diz que parcela excedente "aparece em Pendências como 'Parcela além do total'". Esse texto **não existe** nesta fórmula. Pode vir de outra coluna da view Pendências (ex.: `Status da Parcela Dívida`); **confirmar**.
- O nome do template padrão, "Nova despesa", não está na regex. Um lançamento que mantém esse título passa como "com descrição".
- **Ponto positivo:** `"⚠️ Sem mês"` funciona como detector de falha da automação (TEC-001). Se o comprador estiver num plano em que a automação não roda, todo lançamento acusa. Vale citar isso em "Problemas comuns".
- **Severidade:** Média. **Confiança:** Alta (leitura da fórmula). **Esforço:** P.
- **Recomendação:** acrescentar, nesta ordem de prioridade: Sem tipo; Sem cartão; Sem valor; Sem data; Sem categoria; Sem responsável; Cartão de outra pessoa sem "Devo para"; Investimento sem meta; Sem mês. Incluir "Nova despesa" na regex.
- **Aceite:** um lançamento de teste para cada lacuna aparece na view Pendências com a mensagem certa.

### FORM-003 — Fechamento nos dias 29 a 31 em meses curtos
- **Evidência:** a regra `date(d) >= F` compara com o número do dia. Com fechamento no dia 31, uma compra em 30/04 (último dia do mês) fica na fatura de abril, mas o banco fecharia em 30/04 e a mandaria para a próxima. O mesmo vale para fevereiro com fechamento em 29, 30 ou 31.
- **Severidade:** Baixa (poucos cartões fecham depois do dia 28). **Confiança:** Alta. **Esforço:** P.
- **Recomendação:** usar `min(F, último dia do mês)` ou, mais simples, documentar "use dia de fechamento entre 1 e 28".
- Limitação de modelo, não bug: bancos que adiantam o fechamento em fim de semana ou feriado não são modelados. Basta citar no FAQ.

### FORM-004 — Inconsistência entre os dois rollups do cartão
- **Evidência:** `Dia do Fechamento (cartão)` usa "mostrar original" e por isso precisa de `toNumber(format(...))`. `Dia do Vencimento (cartão)` usa "sum" e já vem número.
- **Consequência:** funciona hoje, mas é frágil: se alguém trocar a agregação, a fórmula muda de comportamento.
- **Severidade:** Baixa. **Esforço:** P. **Recomendação:** os dois como "sum" e remover o `toNumber(format())`. **Aceite:** os 5 casos PASS acima continuam iguais.

### FORM-005 — Taxa de poupança é texto
- **Evidência:** o resultado da fórmula D é do tipo texto ("43%").
- **Consequência:**
  - Não dá para ordenar, filtrar, fazer gráfico nem calcular média anual.
  - Com entradas zero e gasto positivo (nov/26: consumo R$ 700), mostra "–", e o problema some da vista.
- **Severidade:** Baixa. **Esforço:** P.
- **Recomendação:**
  - Fórmula numérica `if(Total Entradas > 0, Sobra do mês / Total Entradas, empty())` com formato %, e o rótulo `🏷` cuida da exibição.
  - Para entradas zero com gasto, mostrar "Sem entradas".

### UX-008 — Nomes diferentes para a mesma coisa no Resumo do mês (visto nos prints)
- **Evidência:** o card "Resumo do mês" mostra "Gastos **Variáveis**" para a propriedade `Pagos à vista`, "Saldo do **Mês**" para `Saldo em caixa` e "Gastos de **consumos**" (erro de concordância).
- **Consequência:** "Variável" já é uma opção de `Natureza` (Fixa/Variável), que é outro conceito. Um aluguel pago no Pix é "Fixa" e aparece em "Gastos Variáveis". Isso gera confusão e pedido de suporte.
- **Severidade:** Média. **Esforço:** P.
- **Recomendação:** um nome só por conceito:
  - "À vista (Pix, débito…)"
  - "Resultado do mês" (ver UX-004)
  - "Gastos de consumo"
- **Aceite:** os rótulos `🏷` iguais aos nomes das propriedades e ao Guia.

## 6. Testes funcionais

Os valores esperados foram calculados a mão a partir dos dados brutos de hoje. **Nenhum valor de fórmula foi observado.** Nenhum caso foi executado.

| ID | Área | Cenário | Esperado | Observado | Status |
|---|---|---|---|---|---|
| FORM-001 | Meses out/26 | Total Entradas | R$ 5.800 | Entradas não consultadas nesta rodada (o guia afirma 5.800) | NOT TESTED |
| FORM-002 | Meses out/26 | Cartões no mês (Streaming 55 + Farmácia 120; o Teste 3x de 10/10 vai para nov) | R$ 175 | Fórmula não legível | NOT TESTED |
| FORM-003 | Meses out/26 | Pagos à vista (1.500 + 320 + 350 + 460) | R$ 2.630 | — | NOT TESTED |
| FORM-004 | Meses out/26 | Gastos de consumo (175 + 2.630 + 500) | R$ 3.305 | — | NOT TESTED |
| FORM-005 | Meses out/26 | Sobra / Taxa de poupança | R$ 2.495 / 43,0% | — | NOT TESTED |
| FORM-006 | Meses out/26 | Saldo em caixa (sobra − aporte) | R$ 1.995 | — | NOT TESTED |
| FORM-007 | Cartão | Compra depois do fechamento (Streaming em 26/09, Nubank fecha 25) | Competência = nov/26 | Pela leitura da fórmula A: 26 ≥ 25 → fecha out → V(5) < F(25) → vence nov | NOT TESTED (lógica confirmada por leitura; ver também os 4 casos do Inter na seção 5.1) |
| QA-015 | Terceiros | Sandbox: "QA-015 Pix pago por terceiro", R$ 100, 18/11/2026, Responsável "QA Terceiro (fictício)" do tipo Terceiro | Gastos de consumo sem os 100; Terceiros no Mês = 100 | Lançamento criado. Valores de rollup e fórmula não são legíveis pelo conector; falta a leitura na tela (ver 6.1) | BLOCKED (falta o print de 11.NOV26 da sandbox) |
| QA-016 | Pendência | Sandbox: "QA-016 Sem tipo de pagamento", R$ 50, sem Tipo, com Categoria, Responsável e Mês | Aparecer na view Pendências ("Sem tipo") | **Não aparece** na view Pendências da sandbox (consulta em 20/11, 21:0x) | **FAIL** (confirma FORM-002) |
| QA-017 | Controle | Sandbox: "QA-017 Controle Pix próprio", R$ 30, Pix, Eu | Fora de Pendências; soma 30 em Pagos à vista e no consumo | Fora de Pendências ✔; valores: aguardam print | PASS parcial |
| QA-018 | Automação | Criar despesa pela API na base duplicada (sandbox) e esperar ~2,5 min | `Mês (auto)` preenchido sozinho | **Continuou vazio** nos 3 lançamentos. Preenchi o mês à mão na sandbox para seguir os testes | **FAIL a classificar**: não se sabe se a automação não foi copiada na duplicação ou se não dispara para criação via API (ver 6.1) |
| QA-001 | Smoke | Duplicar em conta **Free** nova e lançar 1 despesa | `Mês (auto)` preenchido e home com valor | — | BLOCKED (decide COM-001) |
| QA-002 | Smoke | Mesmo teste em conta Plus | Idem | — | BLOCKED |
| QA-003 | Parcelas | Compra 3x no cartão | 3 registros com Categoria, Natureza e Responsável | Dado atual: 2 de 3 sem Categoria | **FAIL por evidência de dado** (não reexecutado) |
| QA-004 | Parcelas | Compra 12x | 12 registros completos, meses consecutivos | — | NOT TESTED |
| QA-005 | Fatura | ✅ Pagar fatura | Só compras com vencimento até hoje ficam Pago | — | NOT TESTED |
| QA-006 | Dívida | Gerar próximas 12 parcelas, dívida em 18x | 18 parcelas, sem "além do total" | — | NOT TESTED |
| QA-007 | Entrada | Planejar o ano | 11 entradas futuras com Responsável | — | NOT TESTED |
| QA-008 | Estorno | Reembolso de terceiro | Entra no caixa, não na renda | — | NOT TESTED |
| QA-009 | Resgate | Entrada categoria Resgate | Não infla renda nem taxa de poupança | — | NOT TESTED |
| QA-010 | Virada de ano | Compra em 28/12 num cartão que fecha dia 25 | Competência = fev do ano seguinte (fatura de jan vence em fev) ou jan, conforme o vencimento | — | NOT TESTED |
| QA-011 | Mês vazio | Mês sem lançamentos | KPIs = 0, sem erro, taxa de poupança sem divisão por zero | — | NOT TESTED |
| QA-012 | Centavos | Despesa de R$ 10,05 × 3 | Total R$ 30,15 sem arredondamento | — | NOT TESTED |
| QA-013 | Celular | Lançamento rápido no app | Conclui em < 1 min | — | NOT TESTED |
| QA-014 | Botões | 7 botões do Lançamento rápido | Cada um cria um lançamento do tipo certo | Rótulos não legíveis | BLOCKED |

**Cobertura de QA: 4 de 25 casos executados na sandbox** (1 FAIL confirmado, 1 FAIL a classificar, 1 PASS parcial, 1 BLOCKED aguardando print). A seção 5.1 traz 10 observações PASS sobre dados existentes.

## 7. Plano de correção priorizado — NÃO EXECUTADO

Cada lote precisa da sua aprovação explícita. Proposta: aplicar numa cópia MASTER, nunca na produção que tem histórico.

| Lote | Prioridade | Achados | Ação | Objeto afetado | Risco | Rollback | Aprovação |
|---|---|---|---|---|---|---|---|
| 0 | P0 | COM-001, TEC-001 | **Você** duplica o link publicado numa conta Free nova e roda QA-001 (eu passo o roteiro) | Nenhum objeto do workspace | Nenhum | — | Pendente |
| 1 | P0 | COM-001/002/003 | Reescrever o material de venda conforme o resultado do Lote 0 | Página "Material de venda" (rascunho privado) | Baixo | Histórico de versões da página | Pendente |
| 2 | P0 | TEC-003, TEC-004 | Apagar a linha vazia, o lançamento sem descrição e as 3 parcelas de teste; atribuir Responsável aos exemplos | 5 registros de Despesas + 11 edições de campo | Baixo (exige autorização específica de exclusão) | Lixeira do Notion (30 dias) | Pendente |
| 3 | P1 | TEC-002 | Incluir Categoria, Natureza e Responsável na ação de gerar parcelas | Automação ou botão de parcelas | Médio (não consigo editar automação via conector; provavelmente você faz manualmente) | Desfazer a ação | Pendente |
| 3b | P1 | FORM-001, FORM-002 | Corrigir `Gastos de consumo`; ampliar `⚠️ Pendência` | 2 fórmulas (Meses, Despesas) | Baixo | Colar a fórmula antiga (transcrita na seção 5.1) | Pendente |
| 4 | P1 | UX-001, UX-002, UX-007, UX-008, COPY-001 | Estratégia de exemplos; Configurações visível; aviso "sem orçamento"; reescrever o callout da dívida | Menu, home, template da dívida | Baixo/Médio | Histórico de versões | Pendente |
| 5 | P2 | UX-003/004/006, TEC-006 | View "Tudo do mês"; renomear "Saldo em caixa"; ocultar Debug; reduzir views; ordenar Parcelas | Views e nomes de propriedade | Médio (nomes usados em fórmulas: renomear propriedade costuma ser seguro no Notion, mas validar) | Renomear de volta | Pendente |
| 6 | P3 | TEC-005, UX-005, PD-001, PD-002 | Redesenho do orçamento, Tipo de lançamento, modo Essencial, MASTER única | Estrutura | Alto | Cópia de segurança antes | Pendente |

## 6.1 Sandbox — registro de escrita e checagens pendentes

**O que foi escrito (só na sandbox; o original não foi tocado):**

| Objeto | Ação | Rollback |
|---|---|---|
| Página "🧪 SANDBOX QA — Dashboard Financeiro (não usar)" | Duplicada de F1 e renomeada | Apagar a página (exige autorização) |
| 👥 Responsáveis (sandbox) | + "QA Terceiro (fictício)", Tipo Terceiro | Apagar o registro |
| Registro de Despesas (sandbox) | + QA-015 (R$ 100), QA-016 (R$ 50) e QA-017 (R$ 30), todos em 18/11/2026, categoria Mercado | Apagar os 3 registros |
| Mesmos 3 registros | `Mês (auto)` = 11.NOV26, preenchido à mão porque a automação não preencheu (QA-018) | Limpar o campo |

Base de nov/26 na sandbox antes dos testes (igual ao original): Cartões 200, À vista 0, Dívidas 500, Consumo 700, Terceiros 0.

**Leitura esperada em 11.NOV26 da sandbox**, para fechar QA-015 e QA-017:

| Campo | Se FORM-001 estiver certo (bug) | Comportamento correto pelo Guia |
|---|---:|---:|
| Pagos à vista | 30 (ou 80, se o lançamento sem tipo contar como à vista) | 30 |
| Terceiros no Mês | 100 | 100 |
| Gastos de consumo | **880** (700 + 100 + 50 + 30) | 730 (ou 780 com o lançamento sem tipo) |
| Saldo em caixa | −880 | −880 (o caixa inclui terceiros, segundo a descrição) |

**Checagem para classificar QA-018:** criar 1 despesa **pela interface do Notion** (botão ou "+ Novo") na sandbox e ver se `Mês (auto)` se preenche.
- Se preencher: a automação foi copiada e só não dispara para criação via API, então não é problema para o comprador.
- Se não preencher: a duplicação não leva a automação, e o comprador que duplicar o template fica sem os totais. Isso é crítico e reforça COM-001 e TEC-001.

## 8. Riscos e limitações
- Sem leitura de fórmula, um erro lógico que não aparece nos dados de exemplo (ex.: virada de ano, mês vazio, centavos) pode existir sem ser detectado.
- A afirmação da v2 de que o mês de exemplo "bate centavo por centavo" compara o recálculo manual com o texto do guia, **não com a saída das fórmulas**. Continua não verificada.
- A hipótese sobre limites do plano Free precisa de confirmação em documentação atual e por teste.

## 9. Parecer de prontidão (digital-product-audit)

| Pergunta | Resposta | Evidência |
|---|---|---|
| O comprador entende o que recebe? | Parcialmente | Material claro, mas com promessas acima do produto (COM-002) |
| Consegue duplicar e configurar sem ajuda? | Não verificado | Depende do plano (COM-001); guia bom, porém oculto (UX-002) |
| Completa a tarefa principal? | Não verificado | Nenhum QA executado |
| Entende o que preencher e quando? | Sim, em boa parte | Configurações Iniciais com 8 passos |
| Sabe recuperar erros comuns? | Parcialmente | Coluna `⚠️ Pendência` + view; sem página de "Problemas comuns" |
| A descrição comercial corresponde ao produto? | **Não** | COM-001, COM-002 |
| Requisitos de plano informados? | **Não / contraditórios** | COM-001 |

**Status: Não pronto** (falha crítica de promessa e risco no fluxo central).

## 10. Próximos passos
1. Você roda o **Lote 0** (teste Free). É o que decide metade do plano.
2. Aprovar quais lotes executar e onde (MASTER nova ou F1).
3. Para destravar FORM-*: colar aqui o código das fórmulas de `Competência`, `Mês de Pagamento (cartão)`, `Gastos de consumo`, `Saldo em caixa`, `Taxa de poupança` e `⚠️ Pendência`, ou mandar prints.
4. Depois de 60 min, a cota de consulta volta e consigo revalidar Entradas, Orçamento, Cartões, Dívidas e Metas.
