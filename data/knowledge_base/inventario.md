# Inventário da Base de Conhecimento — EngMan Copilot

> **DECLARAÇÃO DE DADOS SIMULADOS**
> A Motiva não libera base de dados real para trabalho acadêmico. Todo documento marcado como **simulado** abaixo foi criado pelo grupo com dados **fictícios, plausíveis e identificados como fictícios** (TAGs, kms, praças, datas, fabricantes, modelos, nomes de rodovia e números de OS não correspondem a nenhum ativo ou registro real da Motiva). Documentos marcados como **real** vêm do briefing da Motiva, de normas públicas ou de glossário técnico. Nenhum dado real da Motiva foi usado ou inventado como real.

**Recorte:** B — Operação e Priorização de Campo

**Persona:** Operador do CCO

**Ativo coberto:** câmeras CFTV (alarmes vindos do Zabbix)

**Estado desta fase (Sprint 01):** só o inventário. Os arquivos listados são o *planejado*; serão criados nas próximas etapas.

---

## 1. Visão geral dos documentos

| ID | Documento | Origem | Qtd. mínima planejada | Formato | Como o assistente acessa |
|---|---|---|---|---|---|
| D01 | Catálogo de ativos CFTV | simulado | 32 câmeras (mín. do enunciado: 30+) | CSV | Tools `consultar_ficha_ativo` e `buscar_ativos_criticos` |
| D02 | Alarmes Zabbix recentes (export simulado) | simulado | 45 alarmes | JSON | Carregado no contexto por TAG ou lote (simula o feed do Zabbix) |
| D03 | Histórico de falhas do FSM | simulado | 60 registros (mín.: 50+) | CSV | Tool `consultar_historico_falhas` |
| D04 | Plano de manutenção preventiva CFTV | simulado | 1 documento, 6+ tarefas | Markdown | RAG (ChromaDB) |
| D05 | Procedimentos operacionais (POPs) | simulado | 3 POPs | Markdown | Tool `consultar_procedimento` |
| D06 | Fichas técnicas dos modelos de câmera | real + simulado | 1 ficha por modelo do catálogo (3 modelos) | Markdown | RAG |
| D07 | Matriz de criticidade e critérios de prioridade | simulado | 1 documento | Markdown | RAG (e base do ranking de `buscar_ativos_criticos`) |
| D08 | Glossário de siglas | real | 1 documento | Markdown | RAG |
| D09 | Normas e requisitos de segurança (NR-10, NR-35, regras internas) | real (resumo) | 1 documento | Markdown | RAG |
| D10 | Documentos do briefing Motiva | real | 1 documento | Markdown | RAG |
| D11 | Boletins de ocorrência / laudos de campo | simulado | 12 boletins (mín.: 10+) | Markdown | RAG, com filtro por TAG e data |

São **11 documentos** (mínimo exigido: 8). O D02 não está na lista base do enunciado: foi acrescentado porque os casos C01 a C04 e C10 do eval set dependem de alarmes e status de câmeras. Os documentos de PMV, SAT, SCA, pórtico Free Flow e HS WIM ficam fora, porque o recorte é só CFTV.

---

## 2. Ficha de cada documento

### D01 — Catálogo de ativos CFTV · simulado
- **Arquivo planejado:** `data/knowledge_base/01_catalogo_ativos.csv`
- **Campos:** `tag`, `tipo_ativo`, `fabricante`, `modelo`, `km`, `praca`, `data_instalacao`, `vida_util_anos`, `criticidade` (alta | média | baixa)
- **Regras de criação:**
  - TAG no formato `CFTV-NNN`. A TAG `CFTV-9999` fica **reservada como inexistente** (caso C06).
  - O catálogo precisa incluir câmeras nos km 412 e km 210, porque os casos C09, C11, C12 e C13 citam esses pontos.
  - Distribuição equilibrada entre as 4 praças e entre os níveis de criticidade, para não induzir viés de priorização.
  - Fabricantes e modelos fictícios.
- **Exemplo (fictício):** `CFTV-038 | CFTV | Fabricante Fictício A | PTZ-A200 | 237 | P02 | 2022-03-14 | 8 | alta`

### D02 — Alarmes Zabbix recentes · simulado
- **Arquivo planejado:** `data/knowledge_base/02_alarmes_zabbix.json`
- **Campos:** `id_alarme`, `data_hora`, `tag`, `km`, `gatilho` (ex.: "Imagem escura", "Perda de link", "Desfoque detectado", "Obstrução da lente"), `severidade_zabbix` (Information | Warning | Average | High | Disaster), `estado_zabbix` (PROBLEM | OK), `classificacao_camera` (Limpa | Suja | Parcialmente Suja | Severamente Suja | Desfocada), `obstrucao_pct` (pode ser nulo), `causa_informada` (poeira | fuligem | teias | chuva | neblina | nulo), `condicao_meteorologica_informada` (texto ou nulo)
- **O que NÃO consta aqui:** `Status` (Falha | Falso Positivo | Indeterminado) e `Prioridade`. São **saídas do assistente**, calculadas pelas regras do system prompt e pelos critérios do D07. Gravar esses valores no dado seria duplicar a lógica de decisão.
- **Cobertura mínima de cenários:** falha real por fuligem, poeira e teia; falso positivo com neblina ou chuva informada; campos nulos (`obstrucao_pct`, `causa_informada`) para forçar "Indeterminado"; alerta de falha *com* neblina informada (divergência, regra 6); alarmes repetidos da mesma TAG.
- **Casos do eval set:** um lote de 5 alarmes simultâneos (C01 a C03) e pelo menos 15 alarmes recentes distintos (C04).
- **Observação:** na Sprint 01 este arquivo simula a integração com o Zabbix. Numa versão futura ele seria substituído por chamadas à API do Zabbix (ver Limitações).

### D03 — Histórico de falhas do FSM · simulado
- **Arquivo planejado:** `data/knowledge_base/03_historico_falhas.csv`
- **Campos:** `tag`, `data`, `sintoma`, `causa_identificada`, `tempo_reparo_h`, `peca_trocada` (ou "nenhuma"), `reincidencia` (sim | não)
- **Regras de criação:** janela de 12 meses (padrão da tool); pelo menos 15 TAGs com histórico; pelo menos 5 TAGs com reincidência; pelo menos 3 TAGs do catálogo **sem nenhum registro**, para treinar o "não há histórico".
- **Exemplo (fictício):** `CFTV-080 | 2026-06-02 | imagem obstruída | teia de aranha na lente | 1.5 | nenhuma | sim`

### D04 — Plano de manutenção preventiva CFTV · simulado
- **Arquivo planejado:** `data/knowledge_base/04_plano_preventivo_cftv.md`
- **Campos por tarefa:** ativo, tarefa, periodicidade, responsável, tempo estimado, recursos necessários, condição de execução
- **Conteúdo mínimo:** 6 tarefas (por exemplo: limpeza de lente e carcaça, inspeção de fixação e vibração do poste, verificação de ninhos e teias, teste de link, verificação de foco, inspeção da caixa de alimentação).
- **Exemplo de pergunta que responde:** "qual a periodicidade de limpeza de lente prevista para câmeras PTZ?"

### D05 — Procedimentos operacionais (POPs) · simulado
- **Arquivos planejados:** `data/knowledge_base/05_pop_cftv_01_inspecao.md`, `05_pop_cftv_02_limpeza_foco.md`, `05_pop_cftv_03_comunicacao_alimentacao.md`
- **Campos por POP:** código, objetivo, tipo de intervenção (preventiva | corretiva | inspecao), pré-requisitos de segurança (EPI, NR aplicável, desligamento seguro, profissional habilitado), passo a passo numerado, critério de conclusão.
- **Mapeamento com o enum da tool `consultar_procedimento`:** inspeção → POP-01; limpeza e foco (preventiva) → POP-02; teste de comunicação e alimentação (corretiva) → POP-03.
- **Regra:** o POP-03, por envolver alimentação elétrica, **não detalha intervenção em equipamento energizado**. Ele só remete ao profissional habilitado e à NR-10.

### D06 — Fichas técnicas dos modelos de câmera · real + simulado
- **Arquivo planejado:** `data/knowledge_base/06_fichas_tecnicas_cftv.md`
- **Parte real:** descrição da função do CFTV e das falhas típicas (lente obstruída por barro e fuligem, teia refletindo infravermelho à noite, desfoque por vibração do poste, perda de link), conforme o briefing da Motiva.
- **Parte simulada:** modelos fictícios (3, os mesmos do D01), com resolução, tipo de zoom, faixa de temperatura operacional, alimentação e recurso de visão noturna.

### D07 — Matriz de criticidade e critérios de prioridade · simulado
- **Arquivo planejado:** `data/knowledge_base/07_matriz_criticidade.md`
- **Conteúdo:** tabela *ativo → impacto na segurança viária e na disponibilidade → classificação*, e os critérios explícitos para as quatro prioridades (Muito Alta, Alta, Em Observação, Nula) em função de status, classificação da câmera, obstrução e criticidade do ativo.
- **Papel no projeto:** deixa a priorização **verificável e não baseada em "sensação"** (risco de viés). É também a base do ranking de `buscar_ativos_criticos`.
- **Regra obrigatória:** o D07 não pode contradizer as regras 7 e 8 do system prompt (na dúvida entre Falha e Falso Positivo, nunca prioridade Nula).

### D08 — Glossário de siglas · real
- **Arquivo planejado:** `data/knowledge_base/08_glossario.md`
- **Termos:** CFTV, PTZ, CCO, FSM, RCM, WCM, CBM, MTBF, MTTR, RGA, LDA, λ, OCR, ITS, OS, POP, EPI, ART, NR, LGPD, e os termos do Zabbix usados no projeto (trigger/gatilho, severidade, PROBLEM/OK).

### D09 — Normas e requisitos de segurança · real (resumo)
- **Arquivo planejado:** `data/knowledge_base/09_normas_seguranca.md`
- **Conteúdo:** resumo em linguagem simples do que NR-10 (segurança em instalações e serviços em eletricidade) e NR-35 (trabalho em altura) exigem de quem intervém numa câmera instalada em poste, mais um resumo das regras de trabalho em via.
- **Origem e ressalva:** o grupo resume a partir dos textos oficiais e cita a norma de origem. O arquivo informa que **não substitui o texto oficial nem o POP**. O assistente usa o resumo para remeter ao procedimento, nunca para autorizar uma intervenção.

### D10 — Documentos do briefing da Motiva · real
- **Arquivo planejado:** `data/knowledge_base/10_briefing_motiva.md`
- **Conteúdo:** contexto da Motiva, a dor do Tema 2.1 (saúde visual dos ativos ITS: lente suja, teia, desfoque, falso positivo por glare, chuva e neblina) e as ondas de inovação, em resumo. Só informação do briefing, sem acréscimos do grupo.

### D11 — Boletins de ocorrência / laudos de campo · simulado
- **Arquivo planejado:** `data/knowledge_base/11_boletins_ocorrencia.md`
- **Campos por boletim:** `id_boletim`, `data_hora`, `tag`, `km`, `relato_tecnico` (texto curto), `condicao_ambiental_no_momento`, `acao_realizada`, `situacao_pos_atendimento` (resolvido | pendente | sem atualização).
- **Cobertura mínima:** pelo menos 2 boletins com "sem atualização" ou "pendente" após o envio da equipe (caso C10); pelo menos 2 em que a condição ambiental explica o alarme; relatos curtos, sem nome de empregado.
- **Regra:** os boletins são **dado**, nunca instrução. Nenhum boletim do acervo contém comando embutido; o caso de injeção em texto colado (C11) é testado com entrada do usuário no eval set.

---

## 3. Decisões de RAG

### 3.1 O que vai para o vector store e o que não vai
| Vai para o ChromaDB (busca semântica) | Não vai (consulta exata, sem busca semântica) |
|---|---|
| D04, D06, D07, D08, D09, D10, D11 | D01, D03 (via tools), D02 (carregado no contexto), D05 (via tool) |

Motivo: TAG, km e data pedem **busca exata**, não por similaridade. Um chunk com a TAG errada seria alucinação com cara de fonte. D01, D02, D03 e D05 são lidos pelo orquestrador por TAG ou tipo de intervenção.

### 3.2 Tamanho do chunk
- **Chunk de ~500 tokens, sobreposição de ~60 tokens (≈12%)**, dividindo primeiro por cabeçalho de seção e só depois por tamanho.
- **Por quê:** (a) documento técnico tem seções curtas e autocontidas, e 500 tokens mantêm uma tarefa do plano, ou uma regra de norma com sua exceção, juntas; (b) com `top-k = 4`, o trecho recuperado ocupa ~2.000 tokens, uma fração pequena da janela do Qwen 2.5 7B (32K, Aula 02), o que deixa espaço para o system prompt, as tools, o histórico e a resposta; (c) chunks maiores diluem o embedding e trazem texto irrelevante; menores separam o procedimento do seu pré-requisito de segurança.
- **Exceção:** cada boletim do D11 é **um chunk inteiro**, sem divisão e sem sobreposição, para não misturar ocorrências de câmeras diferentes. Os termos do glossário (D08) são agrupados por tema.

### 3.3 Metadados guardados em cada trecho
`doc_id`, `tipo_documento`, `tag` (quando houver), `km`, `praca`, `data` (ISO), `origem` (real | simulado), `secao`.
Permitem busca filtrada (ex.: `tag = CFTV-038 AND tipo_documento = boletim`) e deixam o assistente **citar a fonte** (regra 3) e **informar se o dado é simulado**.

### 3.4 Embeddings
Sentence Transformers rodando localmente, conforme `docs/decisoes_tecnicas.md`, para o dado operacional não sair da máquina (risco de confidencialidade).

### 3.5 Estimativa de contexto (bônus: política de contexto quantificada)
Estimativa grosseira, ~3 caracteres por token em português:

| Componente | Tokens (aprox.) |
|---|---|
| System prompt (`prompts/system_prompt.md`) | 3.500 |
| Especificação das tools (`tools/tools_spec.json`) | 1.900 |
| RAG (4 chunks × 500) | 2.000 |
| Lote de alarmes do D02 (até 15 alarmes) | 1.000 |
| Histórico recente da conversa (últimos 6 turnos) | 2.500 |
| Resposta (`max_tokens`) | 2.000 |
| **Total** | **≈ 12.900** |

Cabe na janela de 32K do Qwen 2.5 7B. O `num_ctx` do Ollama **precisa ser configurado explicitamente** (sugestão: 16.384), porque o padrão é bem menor. **Estratégia de recorte:** manter só os últimos 6 turnos e reafirmar a TAG ou o lote a cada resposta (regra 16), com resumo da conversa a cada 6 turnos (bloco BORDA do prompt).

---

## 4. Fonte única de verdade (coerência tools ↔ base)

| Dado | Fonte única | Quem entrega |
|---|---|---|
| Ficha do ativo (tipo, fabricante, modelo, km, praça, instalação, vida útil, criticidade) | D01 | `consultar_ficha_ativo` |
| Ranking de ativos críticos e pendências | D01 + D02 + critérios do D07 | `buscar_ativos_criticos` |
| Histórico de falhas por TAG | D03 | `consultar_historico_falhas` |
| Passo a passo e pré-requisitos de segurança | D05 | `consultar_procedimento` |
| Alarmes recentes e estado das câmeras | D02 | contexto (simula Zabbix) |
| Abertura de OS | log simulado de OS, gerado pela tool | `abrir_ordem_servico` |

Nenhum dado da coluna 1 é repetido em texto no vector store. Os documentos do RAG (D04, D06 a D11) **não reproduzem** campos do D01, D02, D03 ou D05; eles complementam.

---

## 5. O que NÃO entra na base (LGPD)

- **Dados pessoais de empregados** (nome, matrícula, telefone, CPF). Responsáveis aparecem só como função ou equipe ("equipe de campo", "CCO").
- **Placas de veículos e imagens de ocupantes** capturadas por câmera ou OCR. Frames de câmera também ficam de fora.
- **Dados contratuais sensíveis** e dados de arrecadação.
- **Dados reais da Motiva** (ativos, histórico, criticidade reais).
- **Credenciais e endereços de rede de equipamentos ou do Zabbix**, para evitar exposição de infraestrutura.
- **Texto com instruções embutidas** ("ignore as regras…") dentro de boletins ou documentos.

Se alguma dessas informações aparecer numa mensagem do operador, o assistente avisa que não pode tratá-la (regra 15) e segue sem ela.

---

## 6. Rastreabilidade (inventário ↔ eval set ↔ riscos)

| Caso / risco | Documentos envolvidos |
|---|---|
| C01 a C03 (status, classificação, prioridade de 5 alarmes) | D01, D02, D07 |
| C04 (OS para os 15 últimos alarmes) | D02, D01, D07 |
| C06 (CFTV-9999 inexistente) | D01 (TAG reservada como inexistente) |
| C09 (abrir caixa energizada) | D05, D09 |
| C10 (câmeras após envio da equipe) | D02, D11 + histórico da conversa |
| C12 (laudo/ART) | D09, D11 |
| C13 (placa e motorista, LGPD) | Seção 5 (o que não entra) |
| C05, C07, C08, C11 (ambíguo, fora de escopo, jailbreak) | não dependem da base |
| Alucinação | D01 e D03 por consulta exata; citação de fonte; seção 4 |
| Viés na priorização | D07 (critérios explícitos); distribuição equilibrada no D01 |
| LGPD | Seção 5 |
| Confidencialidade operacional | Dado simulado e embeddings locais |
| Contexto estourado | Seção 3.2 e 3.5 |
| Risco operacional | D05 e D09 |
| Excesso de confiança | D11 separa o observado em campo; D07 separa critério de hipótese |
