# Avaliação de Arquitetura Multiagente para Identificação de Arquétipos Literários

## Contexto do Pedido

Você é um arquiteto de sistemas de IA especializado em sistemas multiagentes com o Claude.
Seu objetivo é **avaliar e recomendar a melhor arquitetura multiagente** para o sistema
de identificação de arquétipos literários descrito abaixo — considerando que toda a
implementação será feita 100% sobre a API do Claude (Claude Agent SDK / Anthropic API).

Leia com atenção todo o contexto antes de responder. Sua resposta deve ser um documento
de decisão arquitetural com justificativas explícitas para cada escolha.

---

## Sistema Existente (O Que Já Existe)

### Propósito

Sistema de análise de arquétipos literários fundamentado em:
- **Jung** — psicologia analítica (Anima, Animus, Sombra, Si-mesmo, Persona)
- **Campbell** — estrutura narrativa do monomito (jornada do herói)
- **Frye** — padrões literários (modos, ciclos, simbolismo)
- **Neumann** — princípio feminino arquetípico (Grande Mãe)

### Granularidade de Análise

| Modo | Entrada | Saída |
|------|---------|-------|
| `analyze-excerpt` | Trecho com localização exata | Arquétipos no excerto com cadeia de evidência |
| `analyze-document` | Obra completa | Mapa arquetípico da obra com arquétipos centrais/secundários |
| `analyze-corpus` | Coleção de obras | Padrões coletivos, tensões, interpretação do corpus como totalidade |

### Arquétipos Cobertos (10)

Anima · Animus · Sombra · Si-mesmo · Persona · Herói · Trickster · Grande Mãe · Velho Sábio · (Meta: o que é um arquétipo)

Cada arquétipo possui:
- Definição central e fundamentação teórica
- Critérios de evidência em 3 níveis: **ALTA | MÉDIA | BAIXA** confiança
- Diferenciação de arquétipos similares
- Sinais de falso positivo

### Princípios Inegociáveis

1. **Anti-pareidolia** — só nomear arquétipo com evidência de ALTA ou MÉDIA confiança
2. **Rastreabilidade** — todo arquétipo identificado com excertos e localização exata
3. **Reasoning explícito** — cadeia argumentativa da evidência à conclusão
4. **Documentar descartes** — o que foi considerado e rejeitado faz parte da análise

### Estrutura de Arquivos

```
archetypes/
├── training/
│   ├── definitions/      # 10 arquivos .md com critérios por arquétipo
│   └── asserts/          # Exemplos anotados por arquétipo
├── theory/               # Jung, Frye, Campbell (3 arquivos teóricos)
├── inference/
│   ├── corpus/           # Documentos de entrada
│   └── analysis/         # Resultados (schema em _schema.md)
└── prompts/              # Prompts para cada modo de análise
```

---

## Restrição Tecnológica

**Todo o sistema multiagente será implementado 100% sobre o Claude.**

Isso inclui:
- **Orquestração**: Claude como orquestrador (via `claude-opus-4-7` ou `claude-sonnet-4-6`)
- **Agentes especializados**: Claude como subagentes com system prompts distintos
- **Ferramentas**: file reading, structured output, inter-agent communication
- **SDK**: Anthropic Python SDK (`anthropic` package) com suporte ao Agent SDK
- **Comunicação**: tool use (function calling) para comunicação estruturada entre agentes
- **Memória**: context window gerenciado + arquivos de estado em disco
- **Sem frameworks externos**: sem LangChain, LlamaIndex, AutoGen, CrewAI ou similares

---

## O Que Avaliar

### 1. Topologia da Rede de Agentes

Avalie as seguintes topologias, considerando o contexto deste sistema:

#### Opção A — Orquestrador + Agentes de Arquétipo
```
[Orquestrador]
    ├── [Agente: Anima]
    ├── [Agente: Animus]
    ├── [Agente: Sombra]
    ├── [Agente: Si-mesmo]
    ├── [Agente: Persona]
    ├── [Agente: Herói]
    ├── [Agente: Trickster]
    ├── [Agente: Grande Mãe]
    └── [Agente: Velho Sábio]
```
*1 agente por arquétipo, orquestrador agrega os resultados.*

#### Opção B — Orquestrador + Agentes de Escola Teórica
```
[Orquestrador]
    ├── [Agente Jung]        → Anima, Animus, Sombra, Si-mesmo, Persona
    ├── [Agente Campbell]    → Herói (monomito, jornada)
    ├── [Agente Frye]        → Padrões literários, ciclos, simbolismo
    └── [Agente Neumann]     → Grande Mãe, princípio feminino
```
*1 agente por escola teórica, cada um analisa o subconjunto de arquétipos de sua competência.*

#### Opção C — Pipeline Sequencial
```
[Agente Leitura] → [Agente Identificação] → [Agente Validação] → [Agente Síntese]
```
*Cada etapa do protocolo de análise é um agente distinto numa cadeia linear.*

#### Opção D — Arquitetura Hierárquica em 3 Camadas
```
[Orquestrador de Modo]
    ├── [Supervisor Teórico]
    │       ├── [Agente Especialista por Arquétipo]
    │       └── ...
    └── [Agente Anti-Pareidolia] ← revisão independente
```
*Camadas: modo de análise → supervisão teórica → execução especializada.*

#### Opção E — Agente Único com Ferramentas
```
[Agente Claude + tool_use]
    ├── tool: load_archetype_definition(name)
    ├── tool: load_theory_context(school)
    ├── tool: evaluate_confidence(evidence, criteria)
    └── tool: write_analysis(schema, content)
```
*Um agente poderoso com ferramentas que encapsulam o conhecimento arquetípico.*

**Para cada topologia, avalie:**
- Grau de paralelismo possível
- Custo de tokens (chamadas API)
- Coerência interna das análises (risco de contradições entre agentes)
- Aderência ao princípio anti-pareidolia
- Complexidade de implementação com o Claude SDK

---

### 2. Padrões de Coordenação

Avalie qual padrão de coordenação melhor se encaixa para **cada modo de análise**:

#### Padrão "Fan-out / Fan-in"
- Orquestrador distribui o texto para N agentes em paralelo
- Cada agente retorna resultado estruturado
- Orquestrador agrega e resolve conflitos
- *Quando faz sentido neste sistema? Quando não faz?*

#### Padrão "Chain of Thought Distribuído"
- Cada agente produz raciocínio intermediário que o próximo agente usa como entrada
- *Adequado para qual(is) modo(s) de análise?*

#### Padrão "Crítico / Revisor"
- Agente A produz análise
- Agente B (adversarial) tenta refutar com base nos critérios de falso positivo
- Agente A revisa
- *Como implementar o princípio anti-pareidolia como agente distinto?*

#### Padrão "Supervisor com Veto"
- Subagentes fazem identificações
- Supervisor aplica os critérios de confiança e pode vetar identificações de BAIXA confiança
- *Como evitar que o supervisor vire gargalo?*

---

### 3. Gerenciamento de Contexto e Estado

Este sistema tem **conhecimento estático extenso**:
- 10 definições de arquétipos (~500–800 tokens cada)
- 3 arquivos teóricos (~1000–2000 tokens cada)
- Exemplos anotados (training/asserts)

E **estado dinâmico** durante a análise:
- Texto sendo analisado (pode ser um romance inteiro)
- Evidências coletadas até agora
- Arquétipos identificados com raciocínio

Avalie as estratégias:

#### Estratégia A — Contexto Completo no Orquestrador
Orquestrador carrega todas as definições e teorias no context window e repassa para subagentes conforme necessário.
- *Viável? Qual o custo? Qual o risco?*

#### Estratégia B — Carregamento Sob Demanda (tool_use)
Agentes usam ferramentas (`read_file`) para carregar apenas a definição do arquétipo que estão analisando.
- *Como garantir que o agente não ignore critérios relevantes?*

#### Estratégia C — Prompt Caching com Prefill
Usar prompt caching do Claude para os blocos estáticos (definições, teorias), pagando apenas uma vez por sessão.
- *Como estruturar o cache? Quais blocos devem ser cacheados?*

#### Estratégia D — Estado Persistido em Disco
Entre etapas de análise, cada agente serializa seu estado parcial em JSON e o próximo agente desserializa.
- *Para qual(is) modo(s) isso é necessário vs. overhead desnecessário?*

---

### 4. Estratégia de Decomposição de Tarefas para Análise de Documento

A análise de um documento longo (romance, novela) é o caso mais desafiador.
Avalie as abordagens:

#### Chunking Sequencial
- Dividir o texto em chunks de N tokens
- Cada chunk analisado sequencialmente
- Estado de identificações acumulado entre chunks
- *Risco: arquétipo que se manifesta ao longo de toda a obra pode ser fragmentado.*

#### Amostragem Estratégica
- Agente de "leitura primária" varre o documento identificando passagens com alta intensidade simbólica
- Apenas essas passagens são analisadas em profundidade
- *Como calibrar o agente de leitura primária? Critérios de "intensidade simbólica".*

#### Análise Top-Down
- Primeiro: análise estrutural (estrutura narrativa, arco de personagens, modo literário de Frye)
- Depois: hipóteses arquetípicas baseadas na estrutura
- Depois: validação bottom-up com evidências textuais
- *Isso inverte o risco de pareidolia? Como?*

#### Análise Bottom-Up Agregada
- Todos os trechos relevantes analisados independentemente
- Agente de síntese identifica padrões recorrentes como arquétipos centrais
- *Critério de "relevância" para seleção inicial de trechos.*

---

### 5. Handling de Conflitos e Desacordos entre Agentes

Quando dois agentes especialistas chegam a conclusões contraditórias (ex.: Agente Sombra e Agente Trickster ambos identificam o mesmo personagem com ALTA confiança):

Avalie:
- **Protocolo de resolução**: votação, hierarquia, mediação por orquestrador, apresentar ambos
- **Quando conflito é sinal de riqueza analítica** (personagem genuinamente multiarquetípico) vs. **quando é erro**
- **Como documentar o conflito** no schema de saída sem comprometer a clareza da análise

---

### 6. Avaliação de Custo-Benefício por Modo de Análise

Para cada modo (`analyze-excerpt`, `analyze-document`, `analyze-corpus`), estime:

| Critério | Agente Único | Multi-agente Paralelo | Multi-agente Sequencial |
|----------|-------------|----------------------|------------------------|
| Latência | | | |
| Custo de tokens | | | |
| Qualidade analítica | | | |
| Aderência anti-pareidolia | | | |
| Complexidade de impl. | | | |

*Use ordens de grandeza, não números exatos.*

---

### 7. Implementação com o Claude SDK

Considerando que o sistema usa exclusivamente o Claude, avalie:

#### Modelo por Camada
- Orquestrador: `claude-opus-4-7` (raciocínio complexo, tomada de decisão)
- Agentes especialistas: `claude-sonnet-4-6` (análise focada, custo menor)
- Agentes de triagem/leitura: `claude-haiku-4-5` (velocidade, baixo custo)
- *Quando essa estratificação faz sentido? Quando homogeneidade é melhor?*

#### Extended Thinking para Anti-Pareidolia
- O modo de raciocínio estendido (`thinking`) do Claude é adequado para o agente de validação/anti-pareidolia?
- *Qual o custo-benefício de ativar thinking neste contexto?*

#### Structured Output (JSON Mode)
- Como estruturar o output de cada agente como JSON validado para facilitar agregação pelo orquestrador?
- *Propor schema JSON mínimo para comunicação inter-agente.*

#### Prompt Caching na Prática
- Quais blocos de contexto devem ser marcados com `cache_control: {"type": "ephemeral"}`?
- *Definições de arquétipos, teorias, ou o texto sendo analisado?*

---

## Formato da Resposta Esperada

Sua resposta deve ser um **Documento de Decisão Arquitetural (ADR)** estruturado assim:

```
# ADR: Arquitetura Multiagente para Sistema de Arquétipos Literários

## Decisão Recomendada
[Qual arquitetura você recomenda? Uma sentença.]

## Contexto e Forças em Jogo
[O que torna este problema único? Quais restrições dominam a decisão?]

## Opção Recomendada: [Nome]
### Diagrama de Agentes
[Diagrama ASCII da topologia]
### Responsabilidades de cada agente
[Tabela: Agente | Modelo | Responsabilidade | Input | Output]
### Padrão de coordenação
[Como os agentes se comunicam?]
### Estratégia de contexto
[O que cada agente sabe? Como o estado é gerenciado?]

## Alternativas Consideradas e Descartadas
[Para cada opção descartada: por que não?]

## Implementação em 3 Fases
### Fase 1 — MVP (analyze-excerpt funcional)
### Fase 2 — analyze-document com documento longo
### Fase 3 — analyze-corpus com coleções

## Riscos e Mitigações
[Top 3 riscos desta arquitetura e como mitigar]

## Decisões em Aberto
[O que ainda não está claro e precisa de experimentação?]
```

---

## Referências para Embasar a Avaliação

Ao responder, ancore suas recomendações nos seguintes conceitos de design de sistemas multiagentes:

- **Tipos de agentes**: reativo (stimulus-response), deliberativo (goal-based planning), híbrido
- **Coordenação**: emergente vs. centralizada; controle hierárquico vs. descentralizado
- **Decomposição de tarefas**: functional decomposition vs. data decomposition
- **Comunicação**: blackboard, mensagem direta, publish/subscribe
- **Especialização vs. generalização**: trade-off entre agentes altamente especializados e agentes versáteis
- **Coerência global**: como garantir que o resultado final é consistente quando múltiplos agentes contribuem
- **Fail-safe**: o que acontece quando um subagente falha ou retorna resultado inválido?
- **Observabilidade**: como rastrear qual agente tomou qual decisão analítica?

E nos princípios específicos do Claude:
- **Prompt caching** para redução de custo em contextos repetitivos
- **Extended thinking** para raciocínio de alta complexidade
- **Tool use** como mecanismo de comunicação estruturada
- **Subagents via `claude` CLI ou API** para paralelismo real
- **Context window management** em análises longas

---

## Nota Final

A decisão arquitetural deve respeitar os princípios inegociáveis do sistema:

> **Anti-pareidolia, rastreabilidade, reasoning explícito, documentar descartes.**

Esses princípios não são apenas requisitos funcionais — eles são **constraints de design** que devem guiar cada escolha arquitetural. Uma arquitetura que produz análises mais ricas porém menos rastreáveis é uma arquitetura errada para este sistema.
