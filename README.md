# Archetypes — Analisador de Arquétipos Literários

Sistema de análise de arquétipos literários fundamentado em psicologia analítica (Jung), crítica arquetípica (Frye) e mitologia comparada (Campbell). Identifica padrões arquetípicos em textos literários com rigor analítico, evidências textuais explícitas e reasoning rastreável.

## Estrutura

```
archetypes/
├── training/
│   ├── definitions/      # Definições teóricas dos arquétipos
│   └── asserts/          # Exemplos literários anotados por arquétipo
├── theory/               # Teoria avançada (Jung, Frye, Campbell)
├── inference/
│   ├── corpus/           # Documentos a analisar
│   └── analysis/         # Resultados das análises
└── prompts/              # Prompts para cada modo de análise
```

## Arquétipos cobertos

| Arquétipo | Tradição principal | Descrição |
|---|---|---|
| **Anima** | Jung | Princípio feminino na psique masculina |
| **Animus** | Jung | Princípio masculino na psique feminina |
| **Sombra** | Jung | O reprimido, o duplo, o lado oculto |
| **Si-mesmo** | Jung | Totalidade psíquica, meta da individuação |
| **Persona** | Jung | Máscara social, identidade pública |
| **Herói** | Campbell | Jornada de transformação (monomito) |
| **Trickster** | Jung / cross-cultural | Transgressão criativa, liminaridade |
| **Grande Mãe** | Jung / Neumann | Princípio materno cósmico, nutridora e devoradora |
| **Velho Sábio** | Jung / Campbell | Espírito, conhecimento superior, iniciador |

## Como usar

### Analisar um excerto

1. Abra `prompts/analyze-excerpt.md`
2. Cole o excerto no prompt conforme indicado
3. Execute com Claude (ou outro modelo) tendo acesso ao diretório do projeto

### Analisar um documento

1. Coloque o arquivo em `inference/corpus/`
2. Use o prompt em `prompts/analyze-document.md`
3. O resultado vai para `inference/analysis/<obra-autor>.md`

### Analisar um corpus

1. Certifique-se de que os documentos estão em `inference/corpus/`
2. Realize as análises individuais primeiro (recomendado)
3. Use `prompts/analyze-corpus.md` para a síntese coletiva

## Princípios de análise

**Anti-pareidolia** — Arquétipos são identificados apenas quando há evidência textual sólida. O sistema distingue três graus de confiança: *alta* (arquétipo central), *média* (secundário) e *baixa* (especulativo/hipótese).

**Rastreabilidade** — Toda conclusão é ancorada em excertos citados com localização exata.

**Reasoning explícito** — A cadeia argumentativa da evidência à conclusão é sempre documentada.

**Documentação de descartes** — O que foi considerado e rejeitado faz parte do output.

## Fundamentação teórica

- **C.G. Jung** — *The Archetypes and the Collective Unconscious* (CW 9i), *Aion* (CW 9ii)
- **Northrop Frye** — *Anatomy of Criticism* (1957)
- **Joseph Campbell** — *The Hero with a Thousand Faces* (1949)
- **Erich Neumann** — *The Great Mother* (1955)

Detalhes em [`theory/`](theory/index.md).

## Adicionar novos arquétipos ou exemplos

**Nova definição**: crie `training/definitions/<nome>.md` seguindo a estrutura dos existentes (definição central → fundamentação → critérios de evidência → falsos positivos).

**Novo exemplo**: crie `training/asserts/<arquetipo>/<obra-autor>.md` seguindo o template em [`training/asserts/README.md`](training/asserts/README.md).
