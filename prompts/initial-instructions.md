# Projeto: Analisador de Arquétipos Literários

## Contexto e objetivo

Estou construindo um sistema de análise de arquétipos literários com fundamentação
teórica em Carl Gustav Jung (psicologia analítica), Northrop Frye (crítica arquetípica),
Joseph Campbell (monomito), e outros teóricos relevantes.

O objetivo é treinar e operar um modelo de IA capaz de identificar arquétipos
(Anima, Animus, Sombra, Si-mesmo, Herói, Trickster, Grande Mãe, Velho Sábio, etc.)
em textos literários — com rigor analítico e evidências textuais explícitas.

## Estrutura de diretórios

archetypes/
├── training/
│   ├── definitions/      # Definições teóricas de cada arquétipo
│   └── examples/         # Exemplos de obras literárias que ilustram cada arquétipo
├── theory/               # Teoria avançada complementar (Jung, Frye, Campbell, etc.)
├── corpus/               # Documentos a serem analisados
└── analysis/             # Resultados das análises

## Modos de análise

O sistema deve suportar três granularidades:
1. **Excerto** — análise de um trecho específico de um documento
2. **Documento** — análise de um texto individual completo
3. **Corpus** — análise coletiva de todos os documentos, identificando padrões
   arquetípicos recorrentes e dominantes

## Princípios inegociáveis de qualidade analítica

1. **Evidência antes de conclusão**: só nomear um arquétipo quando houver evidência
   textual sólida. Proibido ver "forma em nuvem" — pareidolia analítica.
2. **Rastreabilidade**: todo arquétipo identificado deve ser acompanhado dos excertos
   exatos que sustentam a conclusão.
3. **Reasoning explícito**: o caminho lógico da evidência até a conclusão deve ser
   documentado — qual característica do texto ativa qual critério da definição teórica.
4. **Hierarquia de confiança**: distinguir entre arquétipos centrais (alta evidência),
   secundários (evidência moderada) e especulativos (baixa evidência, sinalizados
   como hipótese).
5. **Contexto literário**: considerar gênero, período, tradição cultural e intenção
   autoral ao interpretar manifestações arquetípicas.

## Output esperado por análise

- Arquétipo(s) identificado(s) com grau de confiança
- Excertos de base (citações exatas com localização no texto)
- Reasoning passo a passo da evidência à conclusão
- Contra-indicações consideradas (o que foi descartado e por quê)
- Referências teóricas utilizadas (qual definição de qual autor embasou a identificação)

## Pedido

Ajude-me a construir o roadmap completo para estruturar essa solução, cobrindo:
1. Conteúdo das definições em training/definitions/ (quais arquétipos, qual estrutura
   interna de cada arquivo)
2. Estrutura dos exemplos em training/examples/ (como organizar e anotar os casos)
3. Estrutura dos arquivos em theory/ (quais obras e conceitos priorizar)
4. Pipeline de análise (como o modelo deve processar corpus → análise)
5. Formato dos arquivos em analysis/ (schema do output)
6. Critérios de validação da qualidade analítica