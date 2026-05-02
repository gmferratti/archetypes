# Archetypes — Instruções para Claude

Este projeto é um sistema de análise de arquétipos literários fundamentado em psicologia analítica (Jung), crítica arquetípica (Frye) e mitologia comparada (Campbell).

## Estrutura do projeto

```
archetypes/
├── training/
│   ├── definitions/      # Definições teóricas de cada arquétipo (10 arquétipos + meta)
│   └── asserts/          # Exemplos literários anotados, organizados por arquétipo
├── theory/               # Teoria avançada: Jung, Frye, Campbell
├── inference/
│   ├── corpus/           # Documentos a serem analisados
│   └── analysis/         # Resultados das análises (schema em _schema.md)
└── prompts/              # Prompts para cada modo de análise
```

## Como realizar uma análise

### Análise de excerto
1. Use o prompt em `prompts/analyze-excerpt.md`
2. Forneça obra, autor, localização e o texto do excerto
3. Salve o resultado em `inference/analysis/`

### Análise de documento
1. Coloque o documento em `inference/corpus/`
2. Use o prompt em `prompts/analyze-document.md`
3. Salve em `inference/analysis/<obra-autor>.md`

### Análise de corpus
1. Certifique-se de que os documentos estão em `inference/corpus/`
2. Preferencialmente, realize análises individuais primeiro
3. Use o prompt em `prompts/analyze-corpus.md`
4. Salve em `inference/analysis/corpus_<nome>.md`

## Princípios inegociáveis de análise

1. **Anti-pareidolia**: só nomear um arquétipo quando os critérios de ALTA ou MÉDIA confiança forem satisfeitos conforme as definições em `training/definitions/`
2. **Rastreabilidade**: todo arquétipo identificado deve ter excertos citados com localização exata
3. **Reasoning explícito**: documentar a cadeia argumentativa da evidência à conclusão
4. **Hierarquia de confiança**: ALTA | MÉDIA | BAIXA — ser explícito sobre o grau de certeza
5. **Documentar descartes**: o que foi considerado e rejeitado faz parte da análise

## Arquétipos cobertos

| Arquivo | Arquétipo |
|---|---|
| `anima.md` | Anima — princípio feminino na psique masculina |
| `animus.md` | Animus — princípio masculino na psique feminina |
| `self.md` | Si-mesmo — totalidade psíquica, meta da individuação |
| `shadow.md` | Sombra — o lado reprimido, o duplo |
| `hero.md` | Herói — a jornada de transformação (monomito) |
| `trickster.md` | Trickster — transgressão criativa, liminaridade |
| `great-mother.md` | Grande Mãe — princípio materno cósmico |
| `wise-old-man.md` | Velho Sábio — espírito, conhecimento superior |
| `persona.md` | Persona — máscara social, identidade pública |
| `what-is-an-archetype.md` | Meta — o que é um arquétipo (fundamento teórico) |

## Hierarquia de autoridade teórica

1. Jung — arquétipos psicológicos (Anima, Animus, Sombra, Si-mesmo, Persona)
2. Campbell — estrutura narrativa heroica (monomito, jornada)
3. Frye — padrões literários (modos, ciclos, simbolismo)
4. Neumann — feminino arquetípico (Grande Mãe)

## Para adicionar novos arquétipos

Crie um arquivo em `training/definitions/<nome>.md` seguindo a estrutura dos existentes:
- Definição central
- Fundamentação teórica (Jung / Frye / Campbell conforme aplicável)
- Características identificadoras
- Manifestações literárias típicas
- Critérios de evidência (alta / média / baixa confiança)
- Diferenciação de arquétipos similares
- Sinais de falso positivo

Para adicionar exemplos, crie `training/asserts/<arquetipo>/<obra-autor>.md` seguindo o template em `training/asserts/README.md`.
