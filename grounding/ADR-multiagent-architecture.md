# ADR: Arquitetura Multiagente para Sistema de Identificação de Arquétipos Literários

**Status**: Proposto  
**Data**: 2026-05-21  
**Autor**: Claude Sonnet 4.6  
**Contexto**: Sistema 100% sobre Anthropic API / Claude Agent SDK

---

## Decisão Recomendada

Adotar uma **arquitetura modo-adaptativa** — topologia de agente único com ferramentas para
`analyze-excerpt`, pipeline de 4 agentes especializados para `analyze-document`, e agente
de síntese sobre análises pré-computadas para `analyze-corpus` — com prompt caching das
definições arquetípicas estáticas e comunicação inter-agente via JSON estruturado.

---

## Contexto e Forças em Jogo

### O que torna este problema singular

A maioria dos sistemas multiagentes distribui uma tarefa computacionalmente pesada por
paralelismo. Este sistema tem um desafio diferente: **distribuir responsabilidade epistêmica**.

O princípio anti-pareidolia não é um filtro de pós-processamento — é uma restrição de processo
que precisa estar presente em cada etapa analítica. Um agente que já "decidiu" que viu uma
Sombra em determinado personagem irá, involuntariamente, selecionar evidências que confirmam
essa decisão. Isso é viés de confirmação, e é o risco dominante neste sistema.

A arquitetura multiagente aqui não serve apenas para paralelismo — serve para criar
**separação epistêmica**: agentes distintos nunca devem ver as conclusões uns dos outros
antes de formarem as próprias hipóteses.

### Restrições que dominam a decisão

| Restrição | Implicação arquitetural |
|-----------|------------------------|
| Anti-pareidolia como princípio inegociável | Agente revisor independente (sem acesso às hipóteses dos especialistas durante a revisão) |
| Rastreabilidade completa | Toda comunicação inter-agente deve ser estruturada e auditável |
| Três modos com complexidades radicalmente distintas | Arquitetura única para todos os modos gera overhead ou insuficiência — topologia por modo |
| 100% Claude | Prompt caching como lever de custo; tool use como protocolo de comunicação |
| Documentos potencialmente longos (romances) | O contexto de um único agente não comporta — decomposição é necessária, não opcional |
| Conhecimento estático volumoso (10 definições + 3 teorias) | Caching agressivo; não repassar definições inteiras via mensagens entre agentes |

---

## Arquitetura Recomendada por Modo

A decisão central é: **a topologia deve variar por modo**, porque os três modos têm
complexidades, volumes de entrada e riscos analíticos fundamentalmente distintos.

### Modo 1: `analyze-excerpt` — Agente Único com Ferramentas

```
┌─────────────────────────────────────────────────────────┐
│  [Analista de Excerto — claude-sonnet-4-6]              │
│                                                         │
│  System prompt (cacheado):                              │
│  ├── 10 definições de arquétipos                        │
│  ├── 3 arquivos teóricos (Jung, Frye, Campbell)         │
│  └── Protocolo anti-pareidolia                          │
│                                                         │
│  Tools disponíveis:                                     │
│  ├── load_archetype_examples(name)                      │
│  └── write_analysis(schema, content)                    │
└─────────────────────────────────────────────────────────┘
```

**Justificativa**: Excertos são curtos — o risco anti-pareidolia não está na falta de
perspectivas, mas na falta de rigor do único analista. Com todas as definições cacheadas
no system prompt, um único agente Sonnet com o protocolo de 4 fases possui contexto
completo para análise rigorosa. Multi-agentes adicionariam latência, custo de coordenação e
risco de contradições artificiais sem ganho analítico real.

A disciplina epistêmica aqui vem do **prompt** (o protocolo 4 fases), não de separação
de agentes. Fase 1 ("leia sem hipótese") e Fase 3 ("verifique critérios") são *modos
cognitivos* do mesmo agente, não agentes distintos.

---

### Modo 2: `analyze-document` — Pipeline de 4 Agentes Especializados

Este é o modo principal e o mais complexo. A arquitetura espelha o protocolo existente de
4 fases, materializando cada fase como um agente distinto com responsabilidade epistêmica
separada.

```
                    ┌──────────────────────┐
                    │   ORQUESTRADOR       │
                    │  (claude-sonnet-4-6) │
                    │                     │
                    │  Carrega o documento │
                    │  Coordena o pipeline │
                    │  Agrega o resultado  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   AGENTE SCANNER     │  ← Fase 1
                    │  (claude-haiku-4-5)  │
                    │                     │
                    │  Leitura sem hipótese│
                    │  Mapeia intensidade  │
                    │  simbólica do texto  │
                    │  → PassagesMap (JSON)│
                    └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    │                      │
                    ▼                      ▼
        ┌───────────────────┐  ┌───────────────────┐
        │  AGENTE JUNG      │  │  AGENTE CAMPBELL  │  ← Fase 2
        │ (claude-sonnet)   │  │  + FRYE + NEUMANN │
        │                   │  │ (claude-sonnet)   │
        │ Anima · Animus    │  │                   │
        │ Sombra · Si-mesmo │  │ Herói (monomito)  │
        │ Persona           │  │ Grande Mãe        │
        │                   │  │ Padrões de Frye   │
        │ → JungFindings    │  │ → NarrFindings    │
        └─────────┬─────────┘  └──────────┬────────┘
                  │                        │
                  └───────────┬────────────┘
                              │
                              ▼
                   ┌──────────────────────┐
                   │  AGENTE REVISOR      │  ← Fase 3
                   │ (claude-sonnet-4-6   │
                   │  + extended thinking)│
                   │                     │
                   │  Recebe APENAS as   │
                   │  evidências brutas,  │
                   │  NÃO as conclusões  │
                   │  dos especialistas  │
                   │                     │
                   │  Aplica critérios   │
                   │  de falso positivo  │
                   │  → ReviewedFindings │
                   └──────────┬───────────┘
                              │
                              ▼
                   ┌──────────────────────┐
                   │  AGENTE SÍNTESE      │  ← Fase 4
                   │ (claude-sonnet-4-6)  │
                   │                     │
                   │  Monta o schema     │
                   │  final, resolve     │
                   │  conflitos, gera    │
                   │  o documento final  │
                   └──────────────────────┘
```

#### Responsabilidades detalhadas

| Agente | Modelo | Input | Output | Acesso ao conhecimento |
|--------|--------|-------|--------|------------------------|
| Orquestrador | sonnet-4-6 | Arquivo do corpus | Coordenação + arquivo final | Schema de output |
| Scanner | haiku-4-5 | Texto completo | `PassagesMap` (JSON) | Marcadores de intensidade simbólica |
| Jung | sonnet-4-6 | Passagens selecionadas | `JungFindings` (JSON) | Definições: Anima, Animus, Sombra, Si-mesmo, Persona + Jung teoria |
| Campbell/Frye | sonnet-4-6 | Passagens selecionadas | `NarrFindings` (JSON) | Definições: Herói, Grande Mãe + Campbell, Frye, Neumann teoria |
| Revisor | sonnet-4-6 + thinking | Evidências brutas (sem labels) | `ReviewedFindings` (JSON) | Todas as definições (critérios de falso positivo) |
| Síntese | sonnet-4-6 | Todos os findings revisados | Documento `.md` final | Schema de output |

---

### Modo 3: `analyze-corpus` — Síntese sobre Análises Pré-computadas

```
┌────────────────────────────────────────────────────────────┐
│  PRÉ-CONDIÇÃO: análises individuais existem em             │
│  inference/analysis/<obra-autor>.md                        │
└────────────────────────────────────────────────────────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  AGENTE EXTRATOR     │
                  │  (claude-haiku-4-5)  │
                  │                     │
                  │  Lê todas as análises│
                  │  Extrai inventário   │
                  │  estruturado por     │
                  │  documento           │
                  │  → CorpusInventory   │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  AGENTE PADRÕES      │
                  │ (claude-sonnet-4-6)  │
                  │                     │
                  │  Identifica padrões  │
                  │  recorrentes em 2+  │
                  │  documentos          │
                  │  → PatternClusters  │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  AGENTE SÍNTESE      │
                  │  CORPUS              │
                  │ (claude-sonnet-4-6)  │
                  │                     │
                  │  Interpreta o corpus │
                  │  como totalidade     │
                  │  Gera documento final│
                  └──────────────────────┘
```

Se análises individuais **não existirem**: o orquestrador executa `analyze-document`
em sequência para cada arquivo do corpus antes de iniciar a síntese. Nunca pular este passo.

---

## Protocolo de Comunicação Inter-Agente

Toda comunicação entre agentes usa **JSON estruturado via tool use**, nunca texto livre.
Isso garante rastreabilidade e auditabilidade de cada decisão analítica.

### Schema: PassagesMap (Scanner → Especialistas)

```json
{
  "document": "string",
  "passages": [
    {
      "id": "p001",
      "location": "Cap. 3, §2",
      "text": "string",
      "intensity_markers": ["numinosity", "transformation", "projected_figure"],
      "structural_function": "liminal_crossing | descent | confrontation | return | other",
      "characters_involved": ["string"]
    }
  ],
  "structural_overview": {
    "narrative_arc": "string",
    "apparent_literary_mode": "romance | comedy | tragedy | irony | unclear"
  }
}
```

### Schema: ArchetypeFindings (Especialistas → Revisor)

```json
{
  "agent": "jung | campbell_frye",
  "document": "string",
  "findings": [
    {
      "archetype": "string",
      "confidence_claimed": "ALTA | MÉDIA | BAIXA",
      "evidence": [
        {
          "passage_id": "p001",
          "quote": "string",
          "reasoning": "string",
          "theoretical_reference": "string"
        }
      ],
      "argument_chain": "string"
    }
  ],
  "considered_and_rejected": [
    {
      "archetype": "string",
      "reason": "string"
    }
  ]
}
```

### Schema: ReviewedFindings (Revisor → Síntese)

```json
{
  "reviewed_findings": [
    {
      "archetype": "string",
      "original_confidence": "ALTA | MÉDIA | BAIXA",
      "reviewed_confidence": "ALTA | MÉDIA | BAIXA | REJECTED",
      "reviewer_reasoning": "string",
      "false_positive_flags": ["string"],
      "multi_archetype_layering": true | false,
      "layering_explanation": "string | null"
    }
  ],
  "conflicts_identified": [
    {
      "archetypes": ["string", "string"],
      "conflict_type": "genuine_layering | false_conflict | one_is_false_positive",
      "resolution": "string"
    }
  ]
}
```

---

## Estratégia de Contexto e Prompt Caching

### O que cachear

O conhecimento estático do sistema (~12.000 tokens) deve ser cacheado como prefixo do
system prompt de todos os agentes especializados. Isso reduz o custo por análise em
aproximadamente 90% para esse bloco.

```python
# Estrutura do system prompt com caching
system = [
    {
        "type": "text",
        "text": ARCHETYPE_DEFINITIONS,  # ~7.000 tokens: 10 arquivos de definição
        "cache_control": {"type": "ephemeral"}
    },
    {
        "type": "text",
        "text": THEORY_FILES,           # ~5.000 tokens: Jung, Frye, Campbell
        "cache_control": {"type": "ephemeral"}
    },
    {
        "type": "text",
        "text": agent_specific_instructions  # instruções específicas do papel — NÃO cachear
    }
]
```

### O que NÃO cachear

- O texto sendo analisado (muda em cada análise)
- As instruções específicas do papel de cada agente (pequenas, sem benefício)
- Os findings dos outros agentes passados como contexto (dinâmico por natureza)

### Carregamento do Agente Scanner

O Scanner (Haiku) **não precisa das definições completas** — precisa apenas dos
marcadores de intensidade simbólica. Seu system prompt é muito menor:

```python
SCANNER_SYSTEM = """
Você é um leitor especializado em identificar passagens com alta intensidade simbólica.
Não analise arquétipos — apenas identifique e classifique passagens.

Marcadores de intensidade simbólica que você deve detectar:
- Numinosidade: momentos de fascínio ou terror desproporcional
- Transformação: personagem qualitativmente diferente antes/depois
- Projeção: reação desproporcional de um personagem a outro (ódio inexplicado, fascínio)
- Liminalidade: travessia de fronteira (física ou simbólica), estados de transição
- Figuras com carga simbólica excessiva: aparecem com mais peso do que sua função narrativa explica
- Descida: movimento em direção ao abismo, escuridão, submundo (literal ou simbólico)
- Retorno: regresso transformado, reconciliação, integração
- Símbolo recorrente: objeto, imagem ou figura que reaparece com carga crescente
"""
```

---

## Resolução de Conflitos entre Agentes

### Tipos de conflito e como tratar

**Tipo 1: Layering legítimo** — Dois agentes identificam o mesmo elemento como manifestação
de arquétipos diferentes, e ambos estão certos.

*Exemplo*: Agente Jung identifica Sombra em Iago (Otelo); Agente Campbell identifica Iago
como parte do caminho de provas do herói Otelo (a tentação do agente sombrio). Ambos são
corretos e complementares — Iago funciona como Sombra projetada *e* como guardião de limiar
na jornada de Otelo.

*Tratamento*: Documentar ambas as identificações com nota de layering. O Agente Revisor deve
flaggar `"multi_archetype_layering": true` e o Agente Síntese apresenta ambas no documento
final com seção "Constelação Arquetípica".

**Tipo 2: Conflito falso** — Os dois agentes estão descrevendo a mesma observação com
vocabulários diferentes.

*Exemplo*: Agente Jung identifica o mentor como Velho Sábio; Agente Campbell identifica o
mesmo personagem como "Ajuda Sobrenatural" (etapa 3 do monomito). São nomes diferentes para
o mesmo arquétipo.

*Tratamento*: O Agente Revisor flagga `"conflict_type": "false_conflict"`. O Agente Síntese
consolida em uma única entrada, referenciando ambas as tradições teóricas.

**Tipo 3: Um é falso positivo** — Dois agentes discordam sobre o mesmo elemento, e um deles
está incorreto.

*Exemplo*: Agente Jung identifica animus com MÉDIA confiança em uma personagem feminina;
Agente Campbell indica que a mesma passagem é simplesmente o "encontro com a deusa" na jornada
do protagonista masculino — o animus identificado é atribuído incorretamente à personagem que
é, na verdade, o arquétipo *para o herói*, não um sujeito arquetípico por si mesma.

*Tratamento*: O Agente Revisor, com thinking ativado, avalia qual identificação tem suporte
textual mais sólido segundo os critérios das definições. Pode rebaixar ou rejeitar a mais
fraca. O descarte é documentado no schema final.

---

## Estratificação de Modelos

| Agente | Modelo | Justificativa |
|--------|--------|---------------|
| Orquestrador | claude-sonnet-4-6 | Coordenação + tomada de decisão; não precisa de Opus |
| Scanner | claude-haiku-4-5 | Tarefa de classificação superficial; velocidade > profundidade |
| Jung | claude-sonnet-4-6 | Análise focada + contexto cacheado = qualidade suficiente |
| Campbell/Frye | claude-sonnet-4-6 | Idem |
| Revisor | claude-sonnet-4-6 + thinking | Extended thinking para raciocínio adversarial rigoroso |
| Síntese | claude-sonnet-4-6 | Formatação estruturada; não requer raciocínio profundo |
| Extrator (corpus) | claude-haiku-4-5 | Leitura e extração estruturada de análises existentes |

**Sobre o uso de Opus**: Reservar `claude-opus-4-7` para análises de complexidade extrema
(corpus de obras de alta densidade simbólica, como Ulysses ou A Divina Comédia) onde o
Agente Revisor precisa de raciocínio mais profundo. Nunca usar Opus para Scanner ou Extrator.

**Sobre Extended Thinking no Revisor**: Ativar `thinking` com `budget_tokens: 8000`. O
Revisor é o único agente em que raciocínio interno longo é justificado — é exatamente ali
que o sistema precisa de deliberação cuidadosa antes de concluir. Para todos os outros
agentes, thinking seria desperdício.

---

## Observabilidade e Auditabilidade

O sistema deve produzir, além do documento de análise final, um **trace de execução**:

```
inference/analysis/
├── <obra-autor>.md              ← documento final (público)
└── traces/
    └── <obra-autor>_trace.json  ← trace completo (debug)
```

O trace contém:
- `scanner_output`: PassagesMap completo
- `jung_raw_findings`: antes da revisão
- `campbell_frye_raw_findings`: antes da revisão
- `reviewer_reasoning`: pensamento completo do Revisor (inclusive thinking tokens, se capturados)
- `conflicts_resolved`: log de decisões do Revisor
- `synthesis_decisions`: o que o Síntese incluiu, rebaixou ou descartou

Este trace não é parte do output analítico final — é a camada de rastreabilidade do processo,
não do resultado.

---

## Alternativas Consideradas e Descartadas

### Opção A — Um agente por arquétipo (10 agentes)

**Por que foi descartada**: Os 10 arquétipos não são domínios independentes — eles estão
teoricamente entrelaçados dentro das escolas. Anima e Animus compartilham toda a teoria de
contrasexualidade junguiana. Sombra e Herói são codependentes na teoria de Campbell (a Sombra
é frequentemente o que o Herói deve integrar). Separar por arquétipo cria agentes que não
têm o contexto teórico necessário para distinguir padrões limítrofes.

Além disso: 10 chamadas paralelas para cada análise de excerto é overhead injustificável para
documentos curtos.

### Opção C — Pipeline Sequencial Simples (4 etapas)

**Por que foi descartada**: A versão estritamente sequencial (Leitura → Identificação →
Validação → Síntese) sem especialização por escola teórica perde a vantagem do paralelismo
disponível. As etapas 2 e 3 (Jung e Campbell/Frye) são genuinamente independentes e podem
rodar em paralelo. Pipeline puro seria mais lento sem ganho de qualidade.

### Opção D — Hierarquia em 3 Camadas

**Por que foi descartada**: Supervisor → Especialista → Revisor adiciona uma camada de
indireção sem responsabilidade clara. O "Supervisor Teórico" seria redundante com o Revisor.
Três camadas de latência seriam o maior gargalo do sistema sem justificativa analítica.

### Opção E — Agente Único com Ferramentas

**Por que foi descartada para `analyze-document`**: Um único agente com contexto de 200K
tokens pode acomodar romances, mas não resolve o problema epistêmico central: o mesmo
agente que gerou uma hipótese na Fase 1 irá confirmá-la na Fase 3. A separação Jung /
Campbell / Revisor não é divisão de trabalho computacional — é divisão de **perspectiva
epistêmica**. Isso é o que a arquitetura recomendada compra.

**Mantida para `analyze-excerpt`** porque o risco de confirmação de hipótese num excerto
curto é menor (há poucos elementos disponíveis), e a disciplina do protocolo de 4 fases
dentro de um único agente é suficiente.

---

## Implementação em 3 Fases

### Fase 1 — MVP: `analyze-excerpt` funcional

**Objetivo**: Validar o protocolo de análise e a estrutura de ferramentas.

```
Entregáveis:
├── Agente único analyze-excerpt com prompt caching
├── tool: load_archetype_examples(name) → retorna asserts do arquétipo
├── tool: write_analysis(variant, content) → escreve no schema correto
├── Testes com 3-5 excertos conhecidos (obras com arquétipos bem estabelecidos)
└── Validação manual: os descartes estão sendo documentados? O reasoning é explícito?
```

**Critério de sucesso**: Analisar um excerto de *O Senhor dos Anéis* (Sombra em Gollum) e
de *Crime e Castigo* (Sombra em Raskolnikov) e produzir outputs que passem na revisão manual
de anti-pareidolia — sem identificações sem evidência sólida.

---

### Fase 2: `analyze-document` com pipeline completo

**Objetivo**: Validar a separação epistêmica e o protocolo JSON inter-agente.

```
Entregáveis:
├── Orquestrador que coordena o pipeline de 4 agentes
├── Scanner com PassagesMap validado por schema JSON
├── Agentes Jung e Campbell/Frye com context caching ativo
├── Revisor com extended thinking e ReviewedFindings
├── Síntese com geração de documento no schema correto
├── Trace de execução em traces/<obra>_trace.json
└── Teste com documento de comprimento médio (~50-100 páginas)
```

**Critério de sucesso**: Análise de *A Metamorfose* de Kafka produz um documento onde:
- Gregor Samsa como herói trágico (Campbell) **e** sua transformação como Sombra (Jung)
  são identificados como layering legítimo, não como conflito
- O Revisor rebaixa pelo menos uma identificação especulativa do pipeline
- O trace mostra exatamente qual agente tomou qual decisão

---

### Fase 3: `analyze-corpus` e observabilidade completa

**Objetivo**: Validar síntese sobre corpus e completar a camada de observabilidade.

```
Entregáveis:
├── Pipeline de análise de corpus (Extrator → Padrões → Síntese)
├── Fluxo condicional: verifica existência de análises individuais antes de iniciar
├── Dashboard de trace (leitura do JSON de trace em formato legível)
└── Teste com corpus de 3-5 obras de um mesmo período ou tradição literária
```

**Critério de sucesso**: Corpus de contos de Machado de Assis produz análise que identifica
padrões arquetípicos recorrentes (Persona fortemente desenvolvida, tensão Sombra/Si-mesmo)
com evidências de pelo menos 3 obras diferentes.

---

## Riscos e Mitigações

### Risco 1: O Scanner Haiku perde passagens críticas

**Probabilidade**: Média. Haiku é rápido e barato, mas pode não capturar nuâncias simbólicas
sutis em prosa densa (Joyce, Woolf, Clarice Lispector).

**Mitigação**: Implementar um threshold de cobertura — o Scanner deve retornar ao menos
`min(10, len(chapters))` passagens. Se o documento for curto (<20 páginas), usar Sonnet
como Scanner. Adicionar ao PassagesMap um campo `scanner_confidence` que o Orquestrador usa
para decidir se roda o Scanner em modo Haiku ou Sonnet.

### Risco 2: Revisor com thinking é gargalo de latência

**Probabilidade**: Alta. Extended thinking adiciona 3-8 segundos ao tempo de resposta.

**Mitigação**: O Revisor é assíncrono — os dois agentes especialistas terminam em paralelo,
o Revisor só começa depois. O gargalo é na Fase 3 do pipeline, não no crítico. Para
análises batch (corpus), rodar em paralelo por documento. Para uso interativo, informar
o usuário que a revisão está em andamento.

### Risco 3: Conflito Jung/Campbell sobre o mesmo personagem causa indecisão na Síntese

**Probabilidade**: Média. Personagens com múltiplas funções arquetípicas são comuns nas
obras mais ricas.

**Mitigação**: O schema `ReviewedFindings` tem `multi_archetype_layering: bool` explícito.
O Agente Síntese tem instrução explícita: layering legítimo não é problema a resolver, é
um resultado a documentar. A seção "Constelação Arquetípica" no schema de saída existe
exatamente para este caso. A falsa pressão de "escolher um arquétipo" deve ser eliminada
das instruções de todos os agentes.

---

## Decisões em Aberto

1. **Chunking de documentos longos**: Para romances >100.000 tokens, o Scanner precisa
   processar em chunks. Definir a estratégia de overlap entre chunks (para não perder
   passagens que cruzam fronteiras de chunk) e como o PassagesMap agrega resultados
   multi-chunk. Recomendação: chunks com 20% de overlap, agente de merge antes de passar
   para os especialistas.

2. **Granularidade do Scanner**: Os marcadores de intensidade simbólica no system prompt
   do Scanner são suficientemente precisos? Precisam de validação empírica com 10-20
   documentos antes de congelar a Fase 2.

3. **Cache invalidation**: Quando uma definição arquetípica for atualizada em
   `training/definitions/`, o cache do Claude precisa ser invalidado. A estratégia de
   versionamento das definições (hash no nome do cache ou timestamp no system prompt)
   precisa ser definida antes da Fase 2.

4. **Corpus sem análises pré-existentes**: O fluxo condicional na Fase 3 (rodar
   `analyze-document` para cada arquivo antes da síntese) pode ser muito lento para
   corpora grandes. Avaliar se análises individuais podem ser paralelizadas com
   controle de concorrência.

---

## Apêndice: Estrutura de Código Sugerida

```
archetypes/
├── src/
│   ├── agents/
│   │   ├── orchestrator.py        # Orquestrador de modo
│   │   ├── scanner.py             # Agente de leitura primária
│   │   ├── jung_specialist.py     # Agente de arquétipos junguianos
│   │   ├── narrative_specialist.py # Agente Campbell/Frye/Neumann
│   │   ├── reviewer.py            # Agente anti-pareidolia com thinking
│   │   └── synthesis.py           # Agente de síntese e output
│   ├── schemas/
│   │   ├── passages_map.py        # Pydantic: PassagesMap
│   │   ├── archetype_findings.py  # Pydantic: ArchetypeFindings
│   │   └── reviewed_findings.py   # Pydantic: ReviewedFindings
│   ├── context/
│   │   ├── cache_builder.py       # Monta system prompts com cache_control
│   │   └── definitions_loader.py  # Carrega training/definitions/ em memória
│   └── tools/
│       ├── file_tools.py          # load_corpus, write_analysis, write_trace
│       └── archetype_tools.py     # load_archetype_examples
└── src/
    └── main.py                    # Entry point: analyze(mode, input_path)
```

---

## Princípio de Fechamento

Esta arquitetura foi projetada para materializar os quatro princípios inegociáveis do sistema
em decisões estruturais, não apenas em instruções de prompt:

| Princípio | Como a arquitetura o materializa |
|-----------|----------------------------------|
| Anti-pareidolia | Agente Revisor com acesso apenas a evidências brutas, sem as hipóteses dos especialistas |
| Rastreabilidade | Comunicação inter-agente via JSON auditável; trace de execução persistido em disco |
| Reasoning explícito | Campo `argument_chain` obrigatório nos schemas; `reviewer_reasoning` sempre presente |
| Documentar descartes | Campo `considered_and_rejected` em `ArchetypeFindings` e `ReviewedFindings`; integrado ao schema de output final |

Uma arquitetura que produz análises mais ricas porém menos rastreáveis é uma arquitetura
errada para este sistema. Cada decisão acima foi avaliada com esse critério como dominante.
