# Schema de Output — Análise Arquetípica

Todo arquivo de análise gerado deve seguir este schema. Três variantes conforme a granularidade.

---

## Variante 1: Análise de Excerto

**Nome do arquivo**: `<obra>_excerto_<n>.md`

```markdown
---
type: excerpt-analysis
work: [título]
author: [autor]
excerpt_location: [localização: cap. X, p. Y, etc.]
date_analyzed: [YYYY-MM-DD]
analyst: [claude-sonnet-4-x ou outra versão]
---

## Excerto analisado

> [texto exato do excerto]

## Arquétipos identificados

### [Nome do arquétipo] — Confiança: [ALTA | MÉDIA | BAIXA]

**Definição aplicada**: [qual aspecto da definição teórica está presente]

**Evidência no excerto**:
- [trecho específico]: [análise de por que é evidência]
- [trecho específico]: [análise de por que é evidência]

**Reasoning**: [cadeia argumentativa: do excerto ao arquétipo]

**Referência teórica**: [Jung CW X / Frye Anatomy p.X / Campbell HWATF p.X]

---

## Arquétipos considerados e descartados

| Arquétipo | Razão do descarte |
|---|---|
| [nome] | [por que não se aplica] |

## Grau de certeza global

[ALTA | MÉDIA | BAIXA] — [justificativa em 1-2 linhas]
```

---

## Variante 2: Análise de Documento

**Nome do arquivo**: `<obra-autor>.md`

```markdown
---
type: document-analysis
work: [título]
author: [autor]
year: [ano]
tradition: [tradição literária]
date_analyzed: [YYYY-MM-DD]
analyst: [modelo]
---

## Resumo analítico

[2-3 parágrafos: o que o documento é e qual é sua estrutura arquetípica central]

## Arquétipo(s) central(is)

### [Nome do arquétipo] — Confiança: ALTA

**Justificativa**: [por que este é o arquétipo central, não secundário]

**Excertos de base**:

> [citação 1]
— [localização]
[análise de por que este excerto é evidência]

> [citação 2]
— [localização]
[análise]

> [citação 3]
— [localização]
[análise]

**Reasoning completo**:
[Cadeia argumentativa estruturada: das evidências isoladas ao padrão, do padrão ao arquétipo central]

**Referências teóricas utilizadas**:
- [Jung: conceito específico, obra, capítulo]
- [Frye: modo, fase, ciclo]
- [Campbell: etapas da jornada presentes]

---

## Arquétipos secundários

### [Nome] — Confiança: MÉDIA

[Mesma estrutura, mais breve]

---

## Arquétipos especulativos

### [Nome] — Confiança: BAIXA

**Sinais presentes**: [o que sugere o arquétipo]
**Por que é especulativo**: [o que falta para confirmar]

---

## Arquétipos considerados e descartados

| Arquétipo | Evidências iniciais | Razão do descarte |
|---|---|---|
| [nome] | [o que parecia indicá-lo] | [por que não se sustenta] |

## Notas sobre contexto literário

[Gênero, período, tradição cultural, intenção autoral — como isso afeta a interpretação]
```

---

## Variante 3: Análise de Corpus

**Nome do arquivo**: `corpus_<nome-do-corpus>_analysis.md`

```markdown
---
type: corpus-analysis
corpus_name: [nome do corpus]
documents_analyzed: [n]
document_list:
  - [doc1]
  - [doc2]
date_analyzed: [YYYY-MM-DD]
analyst: [modelo]
---

## Visão geral do corpus

[Descrição do corpus: o que é, período, tradição, coerência temática]

## Padrões arquetípicos dominantes

### Padrão 1: [Arquétipo ou constelação de arquétipos] — Recorrência: [n/n documentos]

**Descrição do padrão**:
[Como este arquétipo se manifesta de forma recorrente no corpus — variações e constantes]

**Documentos onde aparece**:
| Documento | Manifestação específica | Confiança |
|---|---|---|
| [doc] | [como aparece aqui] | ALTA/MÉDIA |

**Excertos representativos**:
> [excerto do doc1]
— [doc1, localização]

> [excerto do doc2]
— [doc2, localização]

**Análise do padrão coletivo**:
[O que a recorrência significa? É convergência temática consciente, influência literária, ou emergência arquetípica independente?]

---

## Padrões secundários

[Mesma estrutura, mais breve]

---

## Tensões e contradições no corpus

[Arquétipos que se opõem entre documentos, padrões que se contradizem]

## Interpretação do corpus como totalidade

[Síntese: o que o corpus, como conjunto, revela sobre os padrões arquetípicos em operação]

## Limitações da análise

[O que não foi possível determinar, o que exigiria análise mais profunda]
```
