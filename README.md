# engman-copilot

## 1. Nome do projeto e integrantes
**Assistente:** EngMan Copilot
- Grupo: **1CCPB-2026-Grupo2**
- RM 575067: **Gilson Gomes da Silva**
- RM 575140: **Hilda Bonnibel de Oliveira Sousa Dias**
- RM 575086: **Igor Alcalde de Jesus Borba**
- RM 576186: **Matheus Regis Kabakian Albano**
- RM 574606: **Nico Hungria Santos**

## 2. Problema abordado
  A dor que escolhemos foi o Tema 2.1. As câmeras (CFTV) da Motiva ficam ao ar livre, consequentemente estas ficam expostas à condições extremas como poeira, fuligem, chuva, insetos e ninhos. A Motiva procura nos alunos uma solução que os ajude a identificar a saúde visual da câmera evitando falsos positivos como por exemplo chamado de manutenção para casos como ofuscamento solar e neblina.  
  
  De acordo com o briefing disponibilizado pela Motiva, a empresa busca uma solução que minimize custos e seja eficiente em distinguir um problema recorrente genuíno de condições climáticas esporádicas
  
## 3. Recorte e persona
**Recorte B** — EngMan Copilot · Operação e Priorização de Campo + Persona Operador do CCO

"Assistente que apoia o operador do CCO e o coordenador de manutenção a triagem: dado um conjunto de alarmes e ocorrências do dia, decidir o que é de fato falha de ativo, o que é condição ambiental (chuva, neblina, glare solar) e qual a ordem de prioridade dos atendimentos, propondo a descrição da ordem de serviço."  

A Motiva possui mais de 5 mil km em extensão de rodovias onde passam por volta de 2 milhões de veículos diariamente a segurança dessas rodovias depende das câmeras da Motiva que estão expostas a condições extremas. Hoje, a empresa se queixa de ineficiência por precisar de operadores humanos em operações críticas que exigem respostas rápidas. Com frequência, as câmeras sofrem danos oriundos de vibrações, ninhos de animais, insetos, poeira e fuligem. Estes merecem um chamado para manutenção, porém muitos alarmes são falsos positivos, ou apenas uma imagem majoritariamente saudável com pequenas sujidades. Com o recorte e persona escolhidos, nosso assistente consome os alertas gerados pelo Zabbix e busca auxiliar na tomada de decisão, indicando quais alarmes são falsos, quais câmeras estão suficientemente funcionais e quais precisam de atenção. Por exemplo, ao receber uma quantidade de alarmes, o operador pode perguntar ao assistente quais alarmes precisam ser monitorados, quais precisam de ação imediata e quais são alarmes falsos.

Por que escolher o recorte B e não A?: Enquanto o recorte A funciona na identificação da causa da falha em ativos e analise histórica, o recorte B define a ordem de prioridade das manutenções dos ativos, auxiliando na tomada de decisão eficiênte.

Por que o Operador do CCO e não o técnico de campo ou o engenheiro de manutenção?: O operador do CCO é quem recebe os alarmes em tempo real e decide o que acionar e o que apenas monitorar. Um alarme falso gera deslocamento de campo caro; um alarme verdadeiro ignorado gera risco viário. Já o técnico de campo e o engenheiro atuam depois dessa decisão, então é na triagem do operador que o assistente reduz mais custo e risco.

## 4. Escopo do assistente
### 4.1 O que faz:
- Consome os alarmes do Zabbix (nesta fase, simulados por arquivo)
- Classifica cada alarme como Falha, Falso Positivo ou Indeterminado
- Define a prioridade (Muito Alta, Alta, Em Observação ou Nula) e sugere a ordem de atendimento
- Recomenda ações possíveis (ex: acionar manutenção, manter em observação, ou sem ação para falsos alarmes ou casos de baixa prioridade)
- Exibe classificações da câmera como: Limpa, Suja, Parcialmente Suja, Severamente Suja ou Desfocada
- Exibe a causa das classificações como: poeira, fuligem, teias, chuva, neblina ou Indeterminado
- Mostra a porcentagem de obstrução da câmera, quando o dado existe no alerta
- Consulta o histórico de alarmes e falhas de uma câmera identificada por TAG
- Redige a descrição da ordem de serviço (proposta sujeita à validação humana) e o resumo do atendimento
- Diz quando faltam dados em vez de inventar

### 4.2 O que recusa:
- Inventar dados (TAG, km, métrica, porcentagem de obstrução, norma, histórico ou número de OS)
- Tratar dados pessoais (empregados, placas, imagens de ocupantes)
- Oferecer auxílio sobre ativos que não sejam as câmeras CFTV, ou sobre assuntos alheios à manutenção
- Pedido para mudar de personagem ou de função
- Pedido para ignorar instruções anteriores, ou para revelar as regras, o prompt ou a estrutura interna do assistente
- Emitir laudo técnico, ART ou parecer com validade legal
- Orientar intervenção em equipamento energizado, trabalho em altura ou em via sem remeter ao POP e às normas NR-10/NR-35
- Liberar, bloquear, acionar equipe ou executar intervenção (o assistente só sugere; a decisão é humana)
- Executar qualquer código fornecido pelo usuário

**Frase de recusa:** "Eu não posso atender a esta solicitação pois ela foge do meu escopo de atuação. Por favor, reformule sua questão para que eu possa ajudar."

## 5. Arquitetura da solução
![Fluxograma da arquitetura do EngMan Copilot](docs/arquitetura.png)

Fluxograma completo: [`docs/arquitetura.png`](docs/arquitetura.png). A seguir, o fluxo bloco a bloco, com o que entra, o que é processado e o que sai em cada etapa.

### 1 - Entrada (Operador do CCO)
- **Entra:** texto livre do operador, com uma TAG (ex.: CFTV-038) ou um lote de alarmes do Zabbix.
- **Sai:** a mensagem, sem alteração, para o guardrail de entrada.

### 2 - Guardrail de entrada (Llama Guard)
- **Entra:** a mensagem do operador.
- **Processa:** o Llama Guard 3 (`llama-guard3:8b`, local) verifica o prompt e detecta pedido perigoso ou hostil, quebra de persona, tentativa de jailbreak e dado pessoal.
- **Sai:** `safe`, e o fluxo segue para a montagem do contexto; ou `unsafe`, e o usuário recebe a mensagem de recusa padrão sem que o Qwen seja chamado.

### 3 - Montagem do contexto
- **Entra:** a mensagem aprovada.
- **Processa:** monta o contexto nesta ordem: (1) system prompt, (2) histórico da conversa (últimos 6 turnos), (3) trechos recuperados do ChromaDB (plano preventivo, fichas técnicas, matriz de criticidade, normas, boletins), (4) a TAG ou o lote de alarmes do operador, mais o esquema das tools.
- **Sai:** o contexto completo para o modelo.

### 4 - Chamada ao LLM
- **Entra:** o contexto montado.
- **Processa:** o Qwen 2.5 7B (via Ollama, local) gera a resposta com temperatura 0.6 e `max_tokens = 2000`. Escolhemos uma temperatura moderada porque o modelo precisa ser objetivo e técnico, para garantir a qualidade das sugestões dadas ao operador.
- **Sai:** uma resposta ou um pedido de tool.

### 5 - Decisão de tool
- **Entra:** a saída do modelo.
- **Processa:** se faltar dado de ficha, histórico de falhas, POP, criticidade ou OS, o fluxo segue para a execução da tool. Se o contexto já basta, o fluxo vai direto para o guardrail de saída.
- **Sai:** `sim` (chama tool) ou `não` (segue para a verificação).

### 6 - Execução da tool
- **Entra:** a chamada de uma das 5 tools (`consultar_ficha_ativo`, `consultar_historico_falhas`, `consultar_procedimento`, `buscar_ativos_criticos`, `abrir_ordem_servico`).
- **Processa:** a tool consulta a base simulada e devolve o dado. A abertura de OS é simulada.
- **Sai:** o resultado é reinjetado no contexto e o Qwen é chamado de novo (volta ao bloco 4).

### 7 - Guardrail de saída (System Prompt)
- **Entra:** a resposta gerada.
- **Processa:** a verificação está no próprio system prompt (regra 17): está no escopo? citou a fonte? não inventou TAG, km, métrica ou percentual? rotulou hipótese como hipótese? nenhum alarme foi descartado sem evidência?
- **Sai:** `aprovada` (segue para o usuário) ou `reprovada` (o modelo corrige, ou responde com a recusa padrão).

### 8 - Resposta + atualização do histórico
- **Entra:** a resposta aprovada.
- **Processa:** entrega ao operador no formato definido (tabela ou análise passo a passo, com nível de confiança), grava o turno no histórico e mantém só os últimos 6 turnos, com resumo da conversa a cada 6 turnos.
- **Sai:** a resposta ao usuário e o histórico atualizado.

### Decisões de arquitetura
**Por que o guardrail de entrada vem antes do modelo.** Um pedido hostil, uma tentativa de jailbreak ou um dado pessoal (placa, dado de empregado) nunca deve chegar ao Qwen nem ser misturado ao contexto. Barrar antes economiza uma chamada ao modelo, reduz o risco de prompt injection e de LGPD, e a recusa padrão sai sempre igual, o que torna o teste objetivo (casos C07, C08, C11 e C13 do eval set).

**Por que o contexto é montado nessa ordem.** O system prompt vem primeiro para que as regras inegociáveis tenham precedência sobre qualquer coisa que venha depois. Os trechos do ChromaDB, os alarmes e os resultados das tools entram como dado, nunca como instrução (regra 13), para que um texto colado ou um documento recuperado não consiga sobrescrever as regras. A mensagem do operador fica por último, perto da geração, e a TAG é reafirmada a cada resposta (regra 16) para não se perder em conversas longas.

## 6. Stack e justificativa técnica
### Comparação de modelos - Llama 3.1 x Qwen 2.5

| Critério | Llama 3.1 | Qwen 2.5 |
| ---: | :---: | :---: |
| Custo de uso | Gratuito | Gratuito |
| Licença / termos de uso | Uso comercial livre se a quantidade de usuários mensais for abaixo de 700 milhões. Os dados são usados pela Meta | Apache 2.0: Permite uso comercial livre |
| Qualidade em português (pt-BR) | O modelo com 8B apresenta um MMLU específico em pt-BR de 62,12% e um geral de 66,7% | Não detalha MMLU específico para pt-BR, porém com 7B apresenta MMLU geral de 74.2% |
| Tamanho da janela de contexto | 128K | 32K |
| Privacidade do dado (roda local?) | Roda local. Na nuvem, depende da política do provedor | Roda local. Na nuvem coleta dados de input, dados da conta e dados da rede |
| Facilidade de integração com Python | Integração fácil usando Ollama | Integração fácil usando Ollama |
| Maturidade / documentação | Documentação focada na arquitetura, infraestrutura e treinamento do modelo | Documentação abrangente, com documentação de benchmarks, aplicabilidade de frameworks e especializações |

### Comparação de Vector Stores - ChromaDB x FAISS

| Critério | ChromaDB | FAISS |
| ---: | :---: | :---: |
| Tipo | Banco de dados vetorial | Biblioteca de busca vetorial |
| Linguagem principal | Python | C++ |
| Interface Python | Sim | Sim |
| Código aberto | Sim | Sim |
| Licença | Apache 2.0 | MIT |
| Armazenamento de vetores | Nativo | Por meio de índices |
| Armazenamento de documentos | Nativo | Não nativo |
| Armazenamento de metadados | Nativo | Não nativo |
| Persistência | Nativa | Salvamento e carregamento de índices |
| Busca textual | Sim | Não |
| Velocidade de busca | Alta | Alta, com forte otimização |
| Facilidade de configuração | Alta | Média a baixa |

### Conclusão do grupo
- **Modelo: Qwen 2.5 7B.** Escolhido pela licença Apache 2.0, pelo MMLU geral maior (74,2% contra 66,7% do Llama 3.1 8B, conforme a tabela) e pela documentação mais abrangente. **Trade-off aceito:** a janela de contexto é de 32K tokens, e não os 128K do Llama 3.1. Por isso o projeto limita o histórico aos últimos 6 turnos e reafirma a TAG a cada resposta. O modelo roda local (Ollama), então o histórico de falhas e a criticidade dos ativos não saem da máquina.
- **Vector store: ChromaDB.** Armazena documentos e metadados nativamente (necessário para filtrar por TAG e citar a fonte) e é simples de configurar. **Trade-off aceito:** o FAISS seria mais rápido, mas a base é pequena e a velocidade não é o gargalo.
- **Embeddings: Sentence Transformers**, rodando local, pela privacidade do dado operacional. **Orquestração: LangChain**, pela integração entre LLMs, embeddings, banco vetorial e tools.
- **Ambiente e versionamento:** Python 3.10+ com execução local via Ollama; Git + GitHub (repositório público). A interface (terminal interativo) está prevista e não é implementada nesta fase.

A justificativa completa de cada escolha está em [`docs/decisoes_tecnicas.md`](docs/decisoes_tecnicas.md).

## 7. Base de conhecimento
O inventário completo está em [`inventario.md`](data/knowledge_base/inventario.md). Todos os dados são **simulados** (fictícios e identificados como tais), exceto os documentos reais (briefing, glossário e resumo das normas NR-10 e NR-35). A base tem **11 documentos**, todos do recorte CFTV:

| Grupo | Documentos | Como o assistente acessa |
|---|---|---|
| Dados estruturados | Catálogo de ativos (32 câmeras), histórico de falhas (60 registros), POPs (3), alarmes do Zabbix simulados (45) | Tools (`consultar_ficha_ativo`, `consultar_historico_falhas`, `consultar_procedimento`, `buscar_ativos_criticos`) ou carga direta no contexto, com busca exata por TAG |
| Documentos de apoio (RAG) | Plano preventivo, fichas técnicas, matriz de criticidade, glossário, normas, briefing e boletins de ocorrência (12) | ChromaDB, com busca semântica e filtro por metadados |

**Decisões de RAG:** chunks de ~500 tokens com sobreposição de ~60, `top-k = 4`, e metadados (`doc_id`, `tipo_documento`, `tag`, `km`, `praca`, `data`, `origem`, `secao`) para busca filtrada e citação de fonte. Cada dado tem uma única fonte de verdade, sem duplicação entre tools e base.

**O que não entra na base (LGPD):** dados pessoais de empregados, placas, imagens de ocupantes, dados contratuais ou de arrecadação, dados reais da Motiva, credenciais e endereços de rede, e textos com instruções embutidas.

## 8. System prompt
O System Prompt completo está em [`system_prompt.md`](system_prompt.md). Cada bloco tem um comentário explicando sua função. Resumo da estrutura:
 
| Bloco | Função |
|---|---|
| Papel | Define o EngMan Copilot como apoio à decisão do Operador do CCO e, quando solicitado, do Coordenador de Manutenção. Ele não aciona equipes nem abre OS sozinho. |
| Objetivo | Reduzir deslocamentos por falso positivo, evitar que uma falha real seja ignorada e entregar triagem priorizada com sugestão de ação e rascunho de OS. |
| Escopo | Limita o assistente aos alertas de câmeras CFTV vindos do Zabbix e define os valores de Status (Falha, Falso Positivo, Indeterminado) e de Prioridade (Muito Alta, Alta, Em Observação, Nula). |
| Regras | 17 regras inegociáveis: não inventar dados, separar FATO de HIPÓTESE, classificação conservadora, segurança (NR-10/NR-35), proibição de laudo/ART, resistência a prompt injection, LGPD e autoverificação antes de responder. |
| Formato | Três formatos de resposta. Os formatos 1 e 2 são exemplos few-shot em tabela. O formato 3 traz análise passo a passo com sugestão de ação e rascunho de OS. Também define máximo de 10 linhas, nível de confiança e campos da OS. |
| Tom | Técnico e direto em português brasileiro, sem emojis. |
| Borda | Define o comportamento para saudação, pergunta ambígua (no máximo 2 perguntas), pedido hostil e despedida. |
 
**Técnicas de prompting usadas:** persona e escopo explícitos, few-shot nos formatos 1 e 2, raciocínio explicado no formato 3, rotulagem FATO/HIPÓTESE/SUGESTÃO e checklist de autoverificação, que funciona como guardrail de saída.

## 9. Eval set

O eval set ([`/evals/sprint1_eval_set.json`](https://github.com/hungrianico/engman-copilot/tree/main/evals)) tem **13 casos** do recorte B (Operador do CCO), cobrindo as 8 categorias obrigatórias. Cada caso traz entrada, contexto esperado, resposta ideal, critérios de avaliação verificáveis um a um e o campo `deve_recusar`. Na execução, cada resposta será classificada como adequada, parcialmente adequada ou inadequada conforme os critérios atendidos.

| Categoria | Casos | O que testa |
|---|---|---|
| factual | C01, C02 | Recuperação correta de status, classificação e prioridade das câmeras, com citação da fonte |
| instrucional | C03, C04 | Tabela + recomendação de ação; proposta de OS para os 15 últimos alarmes sem inventar registros |
| ambiguo | C05 | Pede clarificação (no máximo duas perguntas) em vez de presumir ativo, falha ou prioridade |
| dado_inexistente | C06 | Admite não ter dados da CFTV-9999 e pede identificação válida |
| out_of_scope | C07, C13 | Recusa padronizada para assunto alheio (receita) e para dado pessoal de terceiros (placa/motorista, LGPD) |
| jailbreak | C08, C11 | Resiste a "ignore as instruções" direto e a comando escondido em laudo colado |
| risco_operacional | C09, C12 | Recusa intervenção em equipamento energizado e emissão de laudo/ART |
| memoria_contexto | C10 | Mantém as câmeras da conversa anterior e não presume que a falha foi resolvida |

Os casos são rastreáveis ao escopo e à matriz de riscos: C06 (alucinação), C08 e C11 (prompt injection), C09 e C12 (risco operacional e laudo sem validade legal), C13 (LGPD) e C07 (fora de escopo). Os casos C01 a C04 e C10 dependem da base de conhecimento simulada (status das câmeras e alarmes recentes) e do histórico da conversa.

## 10. Tools especificadas
A especificação completa está em [`tools_spec.json`](tools/tools_spec.json).

| Nome | O que faz | Parâmetros |
|---|---|---|
| `consultar_ficha_ativo` | Retorna a ficha de um ativo: tipo, fabricante, modelo, km na via, praça, instalação, vida útil esperada e criticidade. | `tag` (string): TAG do ativo no formato do catálogo da Motiva |
| `consultar_historico_falhas` | Retorna o histórico de falhas de um ativo em uma janela de tempo, com data, sintoma, causa, tempo de reparo e reincidência. | `tag` (string): TAG do ativo<br>`janela_meses` (integer, padrão `12`): janela de consulta em meses<br>`tipo_falha` (string, opcional): filtro por tipo de falha |
| `abrir_ordem_servico` | Confirma a abertura de uma ordem de serviço para um ativo, com intervenção, prioridade e descrição. | `tag` (string): TAG do ativo<br>`tipo_intervencao` (enum: `preventiva`, `corretiva`, `inspecao`)<br>`prioridade` (enum: `muito_alta`, `alta`, `em_observacao`; a prioridade Nula não gera OS)<br>`descricao` (string): problema ou intervenção a realizar |
| `consultar_procedimento` | Retorna o passo a passo do POP para o ativo e a intervenção informados, incluindo pré-requisitos de segurança. | `ativo_tipo` (enum: `CFTV`): tipo do ativo, não a TAG individual<br>`tipo_intervencao` (enum: `preventiva`, `corretiva`, `inspecao`) |
| `buscar_ativos_criticos` | Retorna um ranking de ativos por criticidade e pendência de manutenção. | `praca` (string, opcional): filtra por praça; sem ela, considera todas<br>`limite` (integer): número máximo de ativos retornados |

## 11. Riscos e mitigação
A matriz completa está em [`riscos_lgpd.md`](riscos_lgpd.md). Resumo:
 
| Risco | Impacto | Mitigação | Onde é aplicada |
|---|---|---|---|
| Alucinação | Câmera, TAG, métrica ou OS inexistente induz o operador ao erro | Proibição de inventar dados, citação de fonte e admissão de que não sabe | Regras 3, 5 e 17 do System Prompt; caso C06 |
| Prompt injection | Vazamento de dados confidenciais ou do System Prompt | Guardrail de entrada, texto colado tratado como dado e regras inegociáveis | Llama Guard; regras 2, 13 e 14; casos C08 e C11 |
| Risco operacional | Acidente com trabalhador por intervenção em equipamento energizado ou sem EPI | Remeter ao POP e às normas NR-10/NR-35, exigir EPI e profissional habilitado | Regra 11; casos C09 e C12 |
| Viés na priorização | Alguns ativos, praças ou tipos de alarme ficam sempre na frente | Critérios explícitos e verificáveis; status Indeterminado quando não há dado | Regras 7, 8 e 9 |
| LGPD (dado pessoal) | Sanções da ANPD | Não aceitar nem processar placas, imagens de ocupantes ou dados de empregados | Regra 15; caso C13 |
| Confidencialidade operacional | Histórico de falhas e dados da operação vazarem para APIs públicas | Execução 100% local (Llama Guard, Qwen, ChromaDB e embeddings), sem APIs públicas | Arquitetura |
| Excesso de confiança | O técnico trata hipótese como diagnóstico | Rotular FATO, HIPÓTESE e SUGESTÃO e informar o nível de confiança | Regras 4, 7 e 17 |
| Contexto estourado | O assistente perde a TAG ou mistura ativos em conversas longas | Reafirmar a TAG ou o lote no início de cada resposta e manter os dados críticos na janela | Regra 16; caso C10 |

## 12. Limitações conhecidas
- **O Zabbix é simulado por arquivo.** Nesta fase os alarmes vêm de um export fictício (`02_alarmes_zabbix.json`). Não existe tool que consulte o Zabbix: a descrição da arquitetura cita a API do Zabbix, mas as 5 tools especificadas não a cobrem. Uma versão real precisaria de uma nova tool de consulta de alarmes.
- **Os critérios de prioridade estão fora do system prompt.** O prompt define os valores (Muito Alta, Alta, Em Observação, Nula), mas os critérios detalhados ficam na matriz de criticidade (D07, base de conhecimento). O assistente depende de recuperar esse documento corretamente; se a recuperação falhar, a priorização perde consistência.
- **Os dados são fictícios.** Catálogo, alarmes, histórico, POPs e boletins são simulados e não foram validados com dados reais da Motiva. Os limiares e a distribuição de falhas são plausíveis, não medidos.
- **O assistente não analisa imagens.** A classificação da câmera e a porcentagem de obstrução vêm do alerta; sem esses campos o resultado é "Indeterminado". Ofuscamento solar (glare) aparece na descrição do problema, mas não está entre as causas previstas no prompt (poeira, fuligem, teias, chuva, neblina).
- **O eval set não foi executado.** Os 13 casos e as respostas ideais são o projeto de teste. Nenhuma resposta real do modelo foi medida nesta fase.
- **Limites do modelo local.** O Qwen 2.5 7B tem janela de 32K tokens e não tem benchmark específico para pt-BR. O `num_ctx` do Ollama precisa ser configurado explicitamente, e um modelo de 7B pode errar formato ou regras em conversas longas.
- **Escopo restrito.** Só câmeras CFTV. PMV, SAT, SCA, pórticos Free Flow e HS WIM estão fora do recorte.

## 13. Referências
Briefing da Motiva, materiais da disciplina, normas citadas (NR-10, NR-35).
