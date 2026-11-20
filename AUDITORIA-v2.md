# Auditoria — Dashboard Financeiro [ATUALIZADO]

Auditoria feita em 20/11/2026, direto no Notion (via conector), na página
**Dashboard Financeiro [ATUALIZADO]** (a versão editada mais recentemente). A
pasta `export/` e o `AUDITORIA.md` deste repositório tratam da versão
**[ANTIGO]**, que hoje está desatualizada.

Nada foi alterado no template.

---

## 0. O que foi e o que não foi verificado

**Verificado diretamente**

- O esquema completo de 12 bases: Despesas, Entradas, Meses, Orçamento do Mês,
  Grupos, Subcategorias, Cartões, Dívidas, Metas, Responsáveis, Menu e Base do
  Sistema.
- Todos os lançamentos de exemplo (21 despesas e 2 entradas), os 2 cartões, os
  3 responsáveis, os 12 grupos, as 77 subcategorias e os 36 meses.
- A configuração (filtros e colunas) de todas as views do dashboard.
- O resultado real das views em modo "view". Os filtros dependem das fórmulas,
  então isso confirma indiretamente: `É mês atual`, `⚠️ Pendência`,
  `Situação` e `Competência`.
- Os textos do Guia Inicial, das Configurações Iniciais, do Orçamento, de
  Despesas, da página da dívida de exemplo e da meta de exemplo.
- Os números do mês de exemplo (out/2026), recalculados a partir dos dados
  brutos.

**Não foi possível verificar**

| O que falta | Por quê |
|---|---|
| Código das fórmulas e seus valores calculados | O conector do Notion devolve só uma referência (`formulaResult://…`), sem o conteúdo |
| Configuração das automações (ex.: a que preenche `Mês (auto)` e a que gera parcelas) | Automações não são expostas pelo conector |
| Rótulos e ações dos 8 botões da página inicial e dos botões de propriedade (`✅ Pagar fatura`, `Gerar próximas 12 parcelas`, `Planejar…`, `Gasto Fixo Mensal`) | Aparecem como blocos "unknown" |
| Visual renderizado (cores, capa, gráfico, celular) | Sem acesso à tela. A análise visual abaixo vem da estrutura dos blocos |
| Valores de orçamento de out/2026 | Bati no limite de consultas SQL do Notion antes de chegar a esse mês |
| Base "Painel de Investimentos" (ligada a Metas) | Está oculta e não foi aberta |

Para fechar essas lacunas: um vídeo de tela (desktop e celular) duplicando o
template numa conta **Free** nova e fazendo 1 entrada, 1 compra parcelada no
cartão, 1 dívida e 1 pagamento de fatura.

---

## 1. Pontos positivos (manter)

1. **O mês da compra no cartão é tratado do jeito certo.** A compra cai no mês
   em que a fatura vence. Conferi as 4 compras de exemplo contra o fechamento e
   o vencimento de cada cartão, e todas caíram no mês correto:
   - Nubank (fecha 25, vence 5): compra em 10/09 → out.
   - Inter (fecha 9, vence 16): compra em 05/10 → out; compra em 10/10 → nov.

   É o diferencial técnico do produto. Muita planilha erra isso.
2. **O mês de exemplo bate centavo por centavo com o guia.** Entradas R$ 5.800;
   consumo R$ 3.305 (cartões 175 + à vista 2.630 + dívida 500); sobra R$ 2.495;
   investimento R$ 500; saldo R$ 1.995. Grupos (Alimentação R$ 780, Moradia
   R$ 1.500 etc.) e responsáveis (Eu R$ 985, Casa R$ 1.820) também batem.
3. **A coluna `⚠️ Pendência` e a view "Pendências" funcionam.** Elas apontaram
   corretamente a linha em branco, a compra no cartão sem cartão e as parcelas
   sem categoria. É um ótimo recurso de autocorreção para iniciante.
4. **O conceito "investir não é gasto"** (Gastos de consumo × Investimentos ×
   Sobra × Taxa de poupança) é correto e raro em templates.
5. **O filtro `É mês atual` funciona.** Hoje o bloco "Este mês" mostra 11.NOV26,
   e a view de Dívidas do mês mostra só a parcela 2/10.
6. **Orçamento em dois níveis (Grupo → Subcategoria), com padrão R$ ou %.** A
   ordem de prioridade está bem documentada na página Orçamento.
7. **As colunas auxiliares estão escondidas na página.** A maior parte das ~30
   colunas internas de Despesas fica em "Automático" e oculta.
8. **As Configurações Iniciais vêm em passos numerados**, com checklist e
   tempo estimado, mais um roteiro "Teste você mesmo". O formato é muito bom.
9. **Vários problemas da auditoria antiga foram corrigidos:**
   - Limite do Nubank preenchido.
   - Crase removida em "Data da Última Parcela".
   - Dívida de exemplo realista.
   - "Configurações Iniciais" escrito certo.

---

## 2. Problemas identificados

Os problemas estão em ordem de impacto na venda.
🔴 = bloqueia a venda · 🟠 = gera suporte ou reembolso · 🟡 = acabamento

### 🔴 P1. Os cálculos dependem de automação, que pode não rodar no plano Free
O mês de cada lançamento (`Mês (auto)`) é preenchido por uma **automação de
banco de dados**, segundo a descrição da própria propriedade. Pelo texto das
Configurações Iniciais, as parcelas 2…N do cartão também parecem nascer de uma
automação. Todos os totais do mês (`Total Entradas`, `Cartões no Mês`,
`Pagos à vista`, `Dívidas no Mês`) são rollups dessa relação.

- **Impacto:** pelo que conheço da política do Notion, automações de banco de
  dados (fora notificação do Slack) são recurso de plano pago. **Confirme isso
  antes de vender.** Se o comprador estiver no plano Free, os lançamentos podem
  ficar sem mês. O dashboard mostraria zero e as compras parceladas não teriam
  as próximas parcelas. A maioria dos compradores de template está no Free.
- **Solução:**
  1. Teste numa conta Free nova.
  2. A base Meses já tem fórmulas paralelas "(pela data)" e uma `Conferência`,
     que calculam pela data via `Base do Sistema` sem depender da automação.
     Use essas fórmulas como fonte de todos os KPIs e deixe a automação como
     bônus.
  3. Para as parcelas, use **botão** (funciona no Free) em vez de automação.
  4. Se a automação continuar obrigatória, diga "requer plano Plus" na página
     de venda.

### 🔴 P2. As parcelas geradas no cartão perdem a Categoria
Em "Teste - 3x", a parcela 01/03 tem Categoria e as parcelas 02/03 e 03/03 estão
sem ela. As duas aparecem na view Pendências.

- **Impacto:** a cada compra parcelada, as parcelas futuras somem do orçamento
  por grupo e do resumo anual. O comprador vê "gastei menos do que gastei" e
  precisa corrigir à mão a cada parcela.
- **Solução:** na automação ou no botão que gera as parcelas, copiar também
  `Categoria`, `Natureza` e `👤 Responsável`. Depois, refazer o teste de 3x e de
  12x.

### 🔴 P3. Os exemplos ficam em out/2026, mas o dashboard só mostra o mês atual
Todos os blocos de "📍 Este mês" filtram por `É mês atual`.

- **Impacto:**
  - O comprador que abrir em 2027 vê um **dashboard zerado** no primeiro
    contato. A demonstração só aparece se ele abrir 10.OUT26 manualmente.
  - Ao mesmo tempo, as parcelas 4/10 a 10/10 do "Exemplo — Empréstimo" (jan a
    jul/2027) **vão aparecer como dívida real** no mês atual dele, até que ele
    apague os exemplos.
  - O produto vendido como 2027 abre mostrando dados de 2026.
- **Solução (escolha uma):**
  - (a) Não enviar dados de exemplo e trocar por um vídeo curto ou prints no
    guia. É o mais simples.
  - (b) Enviar os exemplos num mês fixo de 2027 (ex.: jan/2027), com um botão
    "🧹 Apagar exemplos" que apaga tudo marcado como exemplo.

### 🔴 P4. Lixo de teste no template
- "Nova despesa": uma linha vazia na base de Despesas.
- Despesa sem descrição, com R$ 100 em "Cartão de Crédito" e sem cartão, em
  10/11/2026.
- "Teste - 3x": 3 parcelas, responsável Parceiro(a), categoria "Dívidas" numa
  compra de cartão.
- `Início do controle` em Meu Financeiro preenchido com 01/10/2026.
- Relações apontando para páginas apagadas:
  - O cartão Inter liga 15 compras, mas só 4 existem.
  - 11.NOV26 liga uma entrada que já não existe.
  - A Base liga 48 meses, mas só 36 existem.

  Rollups normalmente ignoram páginas na lixeira, então isso é limpeza, não
  bug.
- **Impacto:** o novembro do comprador já nasce com R$ 300 de gastos que não
  são dele.
- **Solução:** apagar esses registros, esvaziar a lixeira e limpar
  `Início do controle` (ou deixar instrução clara para preencher).

### 🟠 P5. O Planejamento e o Alerta nascem vazios e o texto engana
Os 12 grupos estão com `Padrão R$` e `Padrão %` vazios. Os registros de
orçamento que consultei (nov/26, jan/27 e dez/28) também estão vazios. A view
"🚨 Atenção e estouros" retorna 0 linhas hoje.

- **Impacto:** o texto do bloco diz *"Se estiver vazio e o Resumo do mês
  mostrar valores, está tudo dentro do orçamento"*. Sem orçamento definido, isso
  é falso: o alerta fica vazio porque não há limite, não porque o comprador está
  dentro dele. O comprador acha que está tudo bem.
- **Solução:**
  - Incluir ⚪ "Sem orçamento" no filtro do alerta, ou mostrar um aviso "Defina
    seu orçamento" quando `Planejado = 0` e `Gasto > 0`.
  - Ou vir com padrões sugeridos (ex.: Moradia 30%, Alimentação 15%…).

### 🟠 P6. A estrutura de orçamento é frágil e trabalhosa
São 432 registros pré-criados (36 meses × 12 grupos). Neles, `Grupo` é um
**select de texto** que precisa ter o nome idêntico ao título na base Grupos.

- **Impacto:**
  - Renomear um grupo quebra o orçamento em silêncio ("Sem orçamento").
  - Para criar um ano novo, o guia manda criar 12 meses à mão e depois
    duplicar e **renomear 144 linhas** de orçamento.
  - Para criar um grupo novo são 4 passos "avançados".
  - Resultado: muito suporte e alto risco de erro.
- **Solução recomendada:** simplificar.
  - `Grupos de Categoria` já tem `Gasto no mês`, `Padrão R$` e `Padrão %`.
  - Faça o "Planejamento do mês" ser uma **view de Grupos**, com uma fórmula de
    Planejado e Situação.
  - Isso elimina os 432 registros, o select duplicado e o processo de ano novo.
  - Ajuste por mês vira recurso opcional, ou sai.
  - Se quiser manter o histórico por mês, troque o select `Grupo` por uma
    **relação** com Grupos.

### 🟠 P7. Compras no cartão não aparecem em "Lançamentos" no dashboard
A view "Gastos" exclui Cartão, Dívida e Investimento.

- **Impacto:** o iniciante lança a primeira compra no cartão e "ela some". Ela
  só aparece somada em "Fatura do Próx Mês". É o pedido de suporte mais
  provável: "meu lançamento não salvou".
- **Solução:** criar uma view "Tudo do mês" (todos os tipos, com coluna
  `Tipo de Pagamento`) como aba padrão. Na view do cartão, mostrar "vai para a
  fatura de <mês>".

### 🟠 P8. A página da dívida traz instruções erradas (texto da versão antiga)
O callout de "Exemplo — Empréstimo" diz:
- "clique em **Criar 10 parcelas**", mas o botão se chama "Gerar próximas 12
  parcelas".
- "registros na base **Saídas**", mas essa base não existe.
- "**ajustar o campo Mês/Ano manualmente**", mas hoje isso é automático.
- "aba **Pagamentos**", "marque o campo **Status**" e "**Status Geral da
  Dívida**": nenhum dos três existe.

Tem também erros de digitação: "Dìvida", "automaticamnete", "pagas,o".

- **Impacto:** o comprador segue a instrução e não acha os botões e campos
  citados.
- **Solução:** reescrever o callout no template da página de dívida, que é de
  onde ele é copiado para cada dívida nova.

### 🟠 P9. `Tipo de Pagamento` mistura forma de pagamento com tipo de lançamento
As opções são: Cartão de Crédito, **Dívida**, DDA, Pix, Transferência,
**Investimento** e Cartão de outra pessoa.

- **Impacto:**
  - Uma parcela de empréstimo paga por Pix precisa ser lançada como "Dívida".
    Um aporte vira "Investimento" em vez de Pix.
  - "DDA" é jargão bancário.
  - **Faltam Débito, Dinheiro e Boleto**, que são formas comuns. O iniciante
    não sabe o que escolher, e a escolha muda o cálculo.
- **Solução:** renomear para **"Tipo de lançamento"**, com estas opções:
  - Cartão de crédito
  - À vista (Pix, débito, dinheiro, boleto)
  - Parcela de dívida
  - Investimento
  - Cartão de outra pessoa

  Se quiser registrar a forma exata, use um campo opcional separado.

### 🟠 P10. "Saldo em caixa" não é saldo
O nome sugere quanto dinheiro existe na conta. Na verdade é o **resultado do
mês** (entrou − saiu). Não existe saldo inicial nem acumulado, e não existem
contas bancárias.

- **Impacto:** o comprador compara com o extrato do banco, os valores não
  batem e ele conclui que o sistema está errado.
- **Solução:** renomear para "Resultado do mês" ou "Fluxo do mês". Opcional:
  um "Saldo acumulado" (saldo inicial + soma dos resultados).

### 🟠 P11. A automação é complexa demais para o perfil do comprador
Cada despesa tem perto de 30 propriedades auxiliares. Existe uma base oculta
"Base do Sistema" que liga todos os registros. Há 7 tipos de pagamento e 9
"Regras importantes" no Guia, incluindo terceiros, reembolsos, compra dividida,
"Devo para" e Balanço.

- **Impacto:**
  - O iniciante sozinho lê regras de casal e de terceiros que não usa.
  - Qualquer coluna editada por engano quebra a cadeia.
  - Com anos de uso, fórmulas que percorrem a Base toda podem ficar lentas,
    principalmente no celular. Isso não foi medido.
- **Solução:**
  - Separar em **Essencial** (entrada, despesa, cartão, orçamento) e
    **Avançado** (terceiros, A receber, A pagar, Responsáveis).
  - Deixar o Avançado recolhido e desligado por padrão.
  - Reduzir as regras do Guia a 3.

### 🟡 P12. Termos de desenvolvedor e views internas visíveis
- A página de cada despesa tem uma seção chamada **"Debug"**, com
  `Competência` visível.
- A view "📋 Todas as despesas" mostra as colunas internas `Vínculo`,
  `Valor Variável` e `Tipo do responsável`.
- A view "🔎 Auditoria de meses" (com Chave e Base) vem junto no produto.
- A base de Despesas tem 21 abas de view, incluindo Jan…Dez, que juntam todos
  os anos.
- **Solução:**
  - Renomear "Debug" para "Conferência", ou ocultar a seção.
  - Tirar as colunas internas das views.
  - Mover a view de auditoria para uma página "Suporte".
  - Reduzir para 5 ou 6 views: Este mês, Por mês, Cartão, Fixas × Variáveis,
    Pendências e Todas.

### 🟡 P13. Menu e navegação
- **Configurações Iniciais está "Oculta" no Menu.** A única entrada é uma
  menção dentro de um toggle fechado do Guia.
- Há espaço no início de 4 nomes do menu: " Cartão de Crédito", " Despesas",
  " Dívidas" e " Entrada". Também sobra espaço em "Reserva de Emergência " e
  nas views "Entradas " e "Dívidas ".
- O ícone ⬜ de Orçamento parece um checkbox vazio.
- **Solução:**
  - Deixar Configurações Iniciais visível no topo até o setup terminar.
  - Ordenar o menu manualmente, sem o truque do espaço.
  - Trocar o ícone de Orçamento por 🟩 ou 🎯.

### 🟡 P14. Ano novo e meses
- O template traz 2026, 2027 e 2028. Para quem compra em 2027, os 12 meses de
  2026 só poluem.
- Para 2029, são 12 meses criados à mão, mais o orçamento (ver P6).
- **Solução:** enviar 2027 a 2030 pré-criados (é barato) e remover 2026.

### 🟡 P15. Investimentos e Metas
- A meta de exemplo tem 1º aporte em 10/01/2026, mas o único aporte é de
  06/10/2026.
- O tipo da meta só tem Viagem, Compra e Reserva.
- Há uma relação oculta com uma base "Painel de Investimentos" que não foi
  auditada. Parece resto de outra versão.
- "Resgate" é lançado como **Entrada**. Não consegui confirmar se `Valor (renda)`
  exclui resgates. Se não excluir, o resgate infla a renda e a taxa de poupança.
- **Solução:**
  - Testar um resgate e conferir as Entradas do mês.
  - Na página de venda, chamar o módulo de "Metas e aportes", não de "controle
    de investimentos" (não há carteira nem rentabilidade).

### 🟡 P16. Cinco cópias do dashboard no workspace
Existem: [ANTIGO], Controle Financeiro 2026, 2026/2027, [2.0] e [ATUALIZADO].

- **Impacto:** risco de publicar ou enviar o link da cópia errada.
- **Solução:** definir uma única "MASTER – não editar", arquivar as outras e
  duplicar a MASTER para gerar o link de venda.

---

## 3. Experiência do usuário (resumo)

| Critério | Avaliação |
|---|---|
| Começar a usar | Médio. O guia é bom, mas fica escondido em toggles e na página oculta. O dashboard abre zerado ou com dados de outro ano (P3) |
| Registrar uma despesa | São ~6 campos: valor, data, tipo, subcategoria (entre 77), responsável e cartão. O botão de lançamento rápido ajuda. A escolha de "Tipo" confunde (P9) |
| Achar o que lançou | Ruim para cartão (P7) |
| Navegar | O Menu funciona, mas a configuração não está nele (P13) |
| Acompanhar a situação | Bons indicadores, mas "Saldo em caixa" engana (P10) e o alerta sem orçamento engana (P5) |
| Iniciante sozinho | Vê regras e blocos de casal e terceiros (A receber, A pagar) que não usa (P11) |

---

## 4. Visual (UI), pela estrutura

Sem acesso ao visual renderizado; confirmar com prints.

- **Hierarquia da home:** Menu + botões → Este mês → Lançamentos → Cartões →
  Metas, A receber, Dívidas, A pagar → Planejamento → **Alerta** → Visão anual.
  O Alerta é a informação mais útil e está no meio da página. Sugestão: logo
  depois do Resumo do mês.
- **Bloco vazio:** há um bloco de 2 colunas vazias logo abaixo do Menu, que só
  ocupa espaço. Remover.
- **Mistura de estilos de ícone:**
  - Emojis de quadrado colorido (🟥🟦🟧🟪⬜).
  - Ícones nativos do Notion em cinza e azul.
  - PNG customizado ("Design_sem_nome_(21).png").

  Escolher um só sistema. O código de cores por módulo é uma boa ideia: aplicar
  também em Entradas e Orçamento.
- **Celular:** as colunas 50/21/29 da primeira linha empilham. No celular o
  Menu, os 3 botões e os outros 4 botões vêm antes de qualquer número.
  - O card "Resumo do mês" tem 20 propriedades (10 rótulos 🏷 + 10 valores):
    fica muito longo.
  - As 21 abas de view em Despesas não cabem na barra do celular.
- **Títulos:** "Dashboard Financeiro **[ATUALIZADO]**" e "📅 Meses
  (automático)" são nomes de trabalho. Renomear antes de duplicar para venda
  (ex.: "Dashboard Financeiro 2027").
- **Callouts cinza em tudo:** quase todos os blocos usam `gray_bg`, então nada
  se destaca. Reservar cor para Alerta e Pendências.

---

## 5. Oportunidades de melhoria (ordem sugerida)

1. **Teste de duplicação no plano Free.** Decide P1 e muda o resto do plano.
2. **Corrigir a cópia de categoria nas parcelas** (P2).
3. **Limpar dados:** testes, linha vazia, lixeira e Início do controle (P4).
   **Decidir a estratégia de exemplos** (P3).
4. **Simplificar o orçamento para uma view de Grupos** (P6). Isso resolve
   junto o P5 e o ano novo (P14).
5. **"Tipo de lançamento" com 5 opções claras** (P9) e **renomear "Saldo em
   caixa"** (P10).
6. **View "Tudo do mês" como padrão** em Lançamentos (P7).
7. **Modo Essencial × Avançado** (P11).
8. **Acabamento:** callout da dívida (P8), "Debug" e views internas (P12),
   espaços e ícones (P13) e uma cópia MASTER (P16).
9. **Refazer o export no repositório** a partir da versão final e arquivar
   `export/` e `AUDITORIA.md`, que descrevem a versão [ANTIGO].
