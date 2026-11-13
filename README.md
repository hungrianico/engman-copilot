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
[Resumo do inventário e como será usada no RAG.]

## 8. System prompt
[Resumo do inventário e como será usada no RAG.]

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
| `abrir_ordem_servico` | Confirma a abertura de uma ordem de serviço para um ativo, com intervenção, prioridade e descrição. | `tag` (string): TAG do ativo<br>`tipo_intervencao` (enum: `preventiva`, `corretiva`, `inspecao`)<br>`prioridade` (enum: `baixa`, `media`, `alta`)<br>`descricao` (string): problema ou intervenção a realizar |
| `consultar_procedimento` | Retorna o passo a passo do POP para o ativo e a intervenção informados, incluindo pré-requisitos de segurança. | `ativo_tipo` (string): TAG do ativo<br>`tipo_intervencao` (enum: `preventiva`, `corretiva`, `inspecao`) |
| `buscar_ativos_criticos` | Retorna um ranking de ativos por criticidade e pendência de manutenção. | `praca` (string, opcional): filtra por praça; sem ela, considera todas<br>`limite` (integer): número máximo de ativos retornados |

## 11. Riscos e mitigação
[Resumo da matriz: risco ? impacto ? mitigação no prompt ou na arquitetura.]

## 12. Limitações conhecidas [isso nao é obrigatorio]
[Onde o assistente ainda falha ou depende de dado que a Motiva precisaria fornecer.]

## 13. Referências
[Briefing da Motiva, materiais da disciplina, normas citadas (NR-10, NR-35), guias de prompting.]



