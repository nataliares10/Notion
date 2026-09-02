# Auditoria de prontidão para venda — DashBoard Financeiro 2027

Base: export do Notion (`export/`), analisado em 2026-09-02.

O template está estruturalmente bem pensado (menu com controle de visibilidade,
rollups, barras de progresso, status por emoji). Os problemas abaixo são de
**acabamento** — o tipo de coisa que o comprador vê nos primeiros 30 segundos e
que gera pedido de reembolso ou avaliação ruim.

Ordenado por impacto na venda.

---

## 🔴 Bloqueadores (corrigir antes de publicar)

### 1. Dados de teste pessoais vazando no template
`Menu Financeiro pjt/Cartão de Crédito/Cartões de Crédito pjt …_all.csv`

A coluna `[NV] Despesa` traz relações apontando para páginas do seu workspace:

- Inter → `TESTE -9 (https://app.notion.com/p/TESTE-9-d894e98d…)`
- Nubank → `TESTE- 25 (https://app.notion.com/p/TESTE-25-e014e98d…)`

O comprador recebe um cartão já "sujo" com registros chamados TESTE, e os links
apontam para páginas privadas suas (vão dar 404 pra ele). Limpar as relações e
apagar as despesas de teste antes de duplicar o template.

### 2. Cartão Nubank abre em estado de erro
Mesmo arquivo. `Limite Total` está **vazio**, o que faz `Status do Limite`
calcular `🚨 Limite excedido` num template zerado.

Primeira impressão do comprador: alarme vermelho sem ter lançado nada. Ou
preencher um limite de exemplo, ou tratar limite vazio na fórmula como
"⚪ Não configurado".

### 3. Seis bases de dados com o nome "Sem título"
`DashBoard Financeiro 2027 …md` referencia cinco views chamadas `Sem título`,
e três CSVs exportados têm esse nome. Aparecem assim na página principal:

| Seção no dashboard | Nome real da base |
|---|---|
| Detalhamento Mensal | Sem título |
| Resumo Mensal | Sem título |
| Cartões de Crédito | Sem título |
| 🟧 (seção sem título nenhum) | Sem título |
| Dívidas | Sem título |

Além de parecer inacabado, quebra a busca do Notion — o comprador não acha nada
digitando o nome. A seção 🟧 nem título tem: pelo conteúdo (`Meta`,
`Resultado Reserva`, `Andamento Meta`) é a de **Metas/Investimentos**.

### 4. Datas de exemplo ainda em 2024–2026 num produto "2027"
| Base | Campo | Valor atual |
|---|---|---|
| Dívidas | Data da 1ª Parcela | 21 de outubro de **2024** |
| Dívidas | Data da Última Parcela | 21 de março de **2026** |
| Cartões (Inter) | Data de Vencimento | 16 de setembro de **2026** |
| Cartões (Nubank) | Data de Vencimento | 5 de setembro de **2026** |
| Metas | Data do 1º Aporte | 10 de janeiro de **2026** |

Vender como "2027" com datas de 2024 é o sinal mais rápido de template
reaproveitado. Reposicionar todos os exemplos dentro de 2027.

---

## 🟠 Erros de texto e nomenclatura

### 5. Erro de português no menu
`Menu Financeiro pjt …_all.csv`: **"Configurações Inicias"** → "Configurações
Iniciais". Está marcada como `Oculta`, mas o comprador vai abrir o banco do menu
para configurar — e vai ver o erro.

### 6. Crase no nome de uma propriedade
Base Dívidas: a coluna chamada ``Data da`Última Parcela`` tem uma **crase (`)**
no lugar do espaço. Nome de propriedade quebrado, aparece assim em toda fórmula
e filtro que a referencie.

### 7. Espaços sobrando em nomes que o comprador vê
Espaço à esquerda ou à direita, que desalinha texto e atrapalha ordenação:

- Menu: `" Dívidas"`, `" Cartão de Crédito"`, `" Despesas"`, `" Planejamento de Gastos"`, `" Entrada"` (todos com espaço à esquerda; só `Investimentos e Metas` está correto)
- Cartões: propriedade `"Valor da Fatura "` (à direita)
- Metas: registro `"Reserva de Emergência "` (à direita)

---

## 🟡 Defaults que passam a mensagem errada

### 8. Dívida de exemplo nasce "🏆 Quitado"
Base Dívidas, registro `Nova Dívida`: `Valor da Dívida` = R$ 0,00, então
`Resultado` = "R$ 0 de R$ 0", `Progresso Barra` = `□□□□□□□□□□ 0%` e o `Status`
cai em **🏆 Quitado**. Uma dívida zerada marcada como quitada não demonstra a
funcionalidade — demonstra um bug. Ou preencher um exemplo real (ex.: R$ 3.600
em 12x), ou proteger a fórmula contra divisão por zero.

Note também a incoerência do próprio exemplo: `Parcela Atual` = **23** com
`Parcelas` = **18** (parcela 23 de 18).

### 9. Coluna `Página Inicial` vazia em todas as 7 linhas do menu
Se o menu serve para navegar, nenhum item aponta para lugar nenhum no export.
Confirmar se a navegação está de fato ligada no template original.

---

## 🔵 Estrutura / polimento

### 10. Colunas internas de fórmula expostas
As bases carregam colunas auxiliares que só interessam ao autor:

- Dívidas: `Data##`, `Parcela Atual##`, `Valor ##`, `Valor da Dívida##`, `Fórmula`, `Gerar 10 Parcelas`
- Cartões: `Limite Disponível##`
- Metas: `Andamento##`, `Objetivo Financeiro##`, `Valor Investido##`, `Contator Conc`

Convenção `##` para uso interno é ótima — mas elas precisam estar **ocultas em
todas as views** do produto vendido. Caso contrário o comprador edita uma sem
querer e quebra o dashboard. Vale um bloco "não altere estas colunas" na
Configurações Iniciais.

### 11. Duas bases referenciadas mas ausentes do export
O `.md` aponta para dois CSVs que não vieram no arquivo:

- `Sem título e5e4e98d4af382ac895d81ba70925d97.csv` → **Detalhamento Mensal**
- `Sem título 5c94e98d4af3828790640195becff719.csv` → **Resumo Mensal**

Também faltam as subpastas de **Despesas**, **Entrada**, **Planejamento de
Gastos** e **Configurações Inicias**, que existem no menu. O export saiu
parcial — provavelmente sem "incluir subpáginas" / sem todas as bases linkadas.
Refazer o export completo antes de qualquer QA final (e antes de arquivar como
versão de venda).

### 12. Link do Guia Inicial aponta pro seu workspace
No `.md`: `[Dasboard Financeiro](https://app.notion.com/p/Dasboard-Financeiro-9a74e98d…)`.
Link absoluto para uma página sua — quebra na cópia do comprador. Deve ser um
link interno relativo dentro do próprio template. (De quebra, "**Dasboard**"
está escrito errado; o correto é "Dashboard", como no título da página.)

---

## Checklist antes de publicar

- [ ] Apagar despesas TESTE e limpar relações `[NV] Despesa`
- [ ] Preencher `Limite Total` do Nubank (ou tratar vazio na fórmula)
- [ ] Nomear as 5 bases "Sem título" + dar título à seção 🟧
- [ ] Reposicionar todas as datas de exemplo em 2027
- [ ] Corrigir "Configurações Inic**ia**is" e "Da**sh**board"
- [ ] Renomear ``Data da`Última Parcela`` (tirar a crase)
- [ ] Tirar espaços sobrando dos 7 nomes listados no item 7
- [ ] Dar valores reais à dívida de exemplo (e corrigir 23/18)
- [ ] Ligar a navegação da coluna `Página Inicial`
- [ ] Ocultar todas as colunas `##` e auxiliares em todas as views
- [ ] Trocar o link do Guia Inicial por link interno
- [ ] Refazer o export completo e reauditar
