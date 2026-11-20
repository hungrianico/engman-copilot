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

## 4. Escopo do assistente
### 4.1 O que faz:
- Consome o Zabbix e os alarmes gerados por ele
- Recomenda possiveis ações (ex: acionar manutenção, manter em observação, ou sem ação para falsos alarmes ou casos de baixa prioridade)
- Dependendo da informação do Zabbix, informa o caso dos ativos conforme solicitação
- Exibe status de alarmes entre falhas reais ou falsos positivios
- Sugere ordem de ação
- Exibe classificações como: Limpa, Suja, Parcialmente suja, Severamente Suja ou Desfocada
- Exibe Causa das classificações como: poeira, fuligem, chuva, neblina
- Diz quando faltam dados ao inves de inventar
- Mostra porcentagem de obstrução da câmera
### 4.2 O que recusa:
- Inventar dados
- Tratar de dados pessoais
- Oferecer auxilio sobre ativos que não sejam as cameras
- Pedido para mudar de personagem
- Pedido para mudar de função
- Pedido para ignorar instruções anteriores
- Pedido para ver histórico/interações anteriores
- Não executa nenhum código dado pelo usuário
**Frase de recusa:** "Eu não posso atender a esta solicitação pois ela foge do meu escopo de atuação"

## 5. Arquitetura da solução
### 1 - Usuário (Operador de CCO)
O usuário que buscamos atender, colaborador no centro de operações da Motiva. O prompt é enviado via texto livre.
### 2 - Guardrail de entrada (Llama-guard)
Um modelo que faz a verificação inicial do prompt do usuário. Caso haja suspeitas de quebra de persona, de privacidade ou de segurança o modelo retorna que o prompt não é seguro.
### 3 - Montagem do contexto
O modelo faz a leitura do System Prompt e retoma o histórico da conversa, dados do ChromaDB e infos sobre as tools.
### 4 - Chamada ao LLM
Contexto é enviado ao Qwen 2.5:7B com temperatura 0.6 e max_tokens = 2000. Uma temperatura mais baixa visto que o modelo precisa ser objetivo e técnico, para garantir a qualidade das sugestões dadas ao operador.
### 5 - Decisão de tool
Ao identificar que o pedido do usuário se encaixa em alguma das tools específicadas, faz o uso delas. Caso não haja informações no contexto para responder ao prompt, o modelo puxa do ChromaDB ou da API do Zabbix.
### 6 - Execução da tool
Ao obter as informações chamadas pela tool, fazemos uma nova chamada ao modelo com um novo contexto montado.
### 7 - Guardrail de saída (System Prompt)
A verificação de saída está no próprio system prompt, o forçando a não inventar dados e manter-se no escopo do assistente.
### 8 - Resposta + atualização do histórico
Após a verificação da resposta, ela é mostrada para o usuário. É gerado um embedding da interação, que é guardado no banco de dados do histórico da conversa.
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
