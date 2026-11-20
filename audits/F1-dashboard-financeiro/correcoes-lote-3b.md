# Lote 3b — Correção de FORM-001 e FORM-002 (aplicar primeiro na SANDBOX)

- Onde aplicar: **somente** "🧪 SANDBOX QA — Dashboard Financeiro (não usar)". O original só depois de todos os testes abaixo darem PASS.
- Como colar: abra a propriedade → Editar fórmula → apague o conteúdo → cole o texto do bloco. O Notion converte cada `prop("…")` em etiqueta. Se aparecer erro de nome, confira o nome exato da propriedade (há emojis como em `👤 Responsável`).
- Rollback: as fórmulas antigas estão logo abaixo de cada nova; basta colar de volta.

---

## 0. Antes de colar — leitura de referência (sandbox, 11.NOV26)

Lançamentos de teste no mês: QA-015 (terceiro, Pix, 100), QA-016 (sem tipo, 50), QA-017 (Pix próprio, 30), **QA-019 (cartão de outra pessoa, 40, sem "Devo para")**, mais a sua "Internet".

Anote, antes de mudar qualquer coisa: **Cartões no Mês, Pagos à vista, Dívidas no Mês, Gastos de consumo, Sobra do mês, Saldo em caixa**, e se QA-019 aparece em Pendências.

Ponto crítico: se `Pagos à vista` **não** aumentou 40 com o QA-019, compras em "Cartão de outra pessoa" não estão em nenhuma das três partes. Nesse caso, **não cole a fórmula 1A**: avise e eu ajusto.

---

## 1. FORM-001 — `📅 Meses → Gastos de consumo`

### 1A. Fórmula nova
```
prop("Cartões no Mês") + prop("Pagos à vista") + prop("Dívidas no Mês")
```
Por quê: as três partes já usam as versões "(próprio)", sem terceiros. Essa é a definição que a própria descrição do campo dá ("cartão + à vista + dívidas"). Investimentos continuam fora.

Fórmula antiga (rollback):
```
prop("Total Despesas") - prop("Investimentos no Mês")
```

### 1B. Só se o `Saldo em caixa` mudar depois da 1A
O caixa **deve continuar incluindo** o que saiu para terceiros (descrição: "tudo que entrou menos tudo que saiu, inclusive gastos de terceiros"). Se, depois da 1A, o Saldo em caixa de 11.NOV26 da sandbox deixar de ser igual a **−(Total Despesas)**, troque a fórmula do `Saldo em caixa` por:
```
prop("Total Entradas") + prop("Reembolsos no Mês") - prop("Total Despesas")
```
Conferência com os exemplos: out/26 → 5.800 + 0 − 3.805 = **1.995** (bate com o Guia). Antes de trocar, copie a fórmula atual do Saldo em caixa para este arquivo (rollback).

---

## 2. FORM-002 — `Registro de Despesas → ⚠️ Pendência`

### Fórmula nova
```
ifs(
  empty(prop("Descrição")) or test(prop("Descrição"), "^(Nova despesa|Novo Gasto|Novo Gasto no Cartão|Investimento|Parcela de Dívida) *$"), "⚠️ Sem descrição",
  empty(prop("Tipo de Pagamento")), "⚠️ Sem tipo de pagamento",
  prop("Tipo de Pagamento") == "Cartão de Crédito" and empty(prop("Cartão de Crédito")), "⚠️ Sem cartão",
  prop("Tipo de Pagamento") == "Cartão de outra pessoa" and empty(prop("Devo para")), "⚠️ Falta o dono do cartão (Devo para)",
  empty(prop("Valor")), "⚠️ Sem valor",
  prop("Valor") < 0, "⚠️ Valor negativo",
  empty(prop("Data da Despesa")), "⚠️ Sem data",
  empty(prop("Categoria")), "⚠️ Sem categoria",
  empty(prop("👤 Responsável")), "⚠️ Sem responsável",
  empty(prop("Mês (auto)")), "⚠️ Sem mês",
  ""
)
```

O que muda em relação à atual:
- "Nova despesa", o nome do template padrão, passa a contar como sem descrição.
- Novas checagens: **Sem tipo**, **Cartão de outra pessoa sem Devo para**, **Valor negativo** e **Sem responsável**.
- Ordem: primeiro o que afeta os totais, por último o mês (que é automático).

Fórmula antiga (rollback):
```
ifs(empty(prop("Descrição")) or test(prop("Descrição"), "^(Novo Gasto|Novo Gasto no Cartão|Investimento|Parcela de Dívida) *$"), "⚠️ Sem descrição", prop("Tipo de Pagamento") == "Cartão de Crédito" and empty(prop("Cartão de Crédito")), "⚠️ Sem cartão", empty(prop("Valor")), "⚠️ Sem valor", empty(prop("Data da Despesa")), "⚠️ Sem data", empty(prop("Categoria")), "⚠️ Sem categoria", empty(prop("Mês (auto)")), "⚠️ Sem mês", "")
```

⚠️ Efeito esperado da linha **Sem responsável**: os exemplos de dívida e de aporte, que não têm responsável (TEC-004), vão passar a aparecer em Pendências. Isso é desejado. **Mas** se algum botão do ⚡ Lançamento rápido não preencher o Responsável, todo lançamento rápido vai nascer com pendência. Teste T6 abaixo. Se acontecer, apague só essa linha da fórmula, ou ajuste os botões.

Deixei de fora de propósito:
- "Investimento sem Meta": Metas são opcionais no guia.
- "Dívida sem vínculo": não sei se as parcelas geradas pelo botão preenchem a relação. Avaliar depois.

---

## 3. Testes de aceite (na sandbox, depois de colar)

| ID | Teste | Esperado |
|---|---|---|
| T1 | 11.NOV26: Gastos de consumo | = Cartões + Pagos à vista + Dívidas (sem os 100 do QA-015) |
| T2 | 11.NOV26: Saldo em caixa | Igual à leitura anterior (inclui terceiros). Se mudou, aplicar 1B |
| T3 | 10.OUT26: Gastos de consumo / Sobra / Saldo / Taxa | 3.305 / 2.495 / 1.995 / 43% |
| T4 | Pendências: QA-016 | "⚠️ Sem tipo de pagamento" |
| T5 | Pendências: QA-019 | "⚠️ Falta o dono do cartão (Devo para)" |
| T6 | Criar 1 despesa por **cada** botão do ⚡ Lançamento rápido, preenchendo o que o botão pede | Nenhuma "Sem responsável" indevida |
| T7 | QA-017 e a sua "Internet" | Fora de Pendências |

Mande um print de 11.NOV26, um de 10.OUT26 e um da view Pendências, e eu registro os resultados no relatório.
