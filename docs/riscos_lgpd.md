## Riscos, ética e LGPD

### Alucinação
<!-- - Inventar TAG, modelo, número de OS, métrica, trecho de norma ou registro de histórico. -->
- DESCRIÇÃO: Alucinação faz que o modelo crie dados não existentes, fazendo com que a resposta seja inútil ou prejudicial, enganando o operador.
- IMPACTO: Invenção de câmera, TAG, métrica ou OS inexistente induz o operador a acionar uma equipe sem necessidade (impacto financeiro) ou a ignorar uma falha real (impacto de segurança viária).
- MITIGAÇÃO: Deve ser mitigado por dar regras claras para não inventar dados, sempre citar suas fontes e admitir que não consegue responder algo.
- APLICAÇÃO: É aplicado na regra 3, 5 e 17 do systemprompt

### Prompt injection
<!-- - "Ignore as instruções" ou comando escondido dentro de um laudo colado. -->
- DESCRIÇÃO: Tentativa de um usuário mal intencionado para contornar burlar o systemprompt e guardrails para obter dados confidenciais, o systempromt, ou tentar comprometer os dados.
- IMPACTO: Risco de vazar dados confidenciais da empresa e clientes, acarretando em possíveis processos.
- MITIGAÇÃO: Deve ser mitigado através de bons guardrails, systemprompt e evalset. Para reconhecer sistematicamente cada tentativa de jailbreak.
- APLICAÇÃO: É aplicado nos casos C08 e C11 do evalset, na arquitetura via o llama_guard, e nas regras 2, 13 e 14 do systemprompt

### Risco operacional
<!-- - Orientar intervenção em equipamento energizado ou em via sem procedimento de segurança. 
-->
- DESCRIÇÃO: Orientar ou recomendar processos perigosos sem indicar como se prevenir ou se proteger, como intervir em equipamento energizado ou ignorar padrões de segurança, gerais e específicos.
- IMPACTO: Ferir ou matar um trabalhador da Motiva por falta de cuidado.
- MITIGAÇÃO: Especificar as normas de segurança necessárias, como a NR-10 e NR-35, POPs e regras de trabalho internas, e exigir que seguir as normas, POPs, regras e usar EPI sempre deve ser especificado e esclarecido para o operador.
- APLICAÇÃO: É aplicado na regra 11 do systemprompt e no caso C09 do evalset

### Viés na priorização
<!-- - Priorizar sempre o mesmo tipo de ativo, região ou praça. -->
- DESCRIÇÃO: Demonstrar viés para certos alarmes, tipos de problemas, status, câmeras, e soluções, dando prioridade esses mesmo que não sejam o necessário.
- IMPACTO: Diminui a eficiência das operações, causando mais falsos positivos, e faz que câmeras com problemas sejam inacessíveis
- MITIGAÇÃO: Usar critérios explícitos e verificáveis para suas decisões.
- APLICAÇÃO: Regras 7, 8 e 9 do system prompt (classificação conservadora, status Indeterminado quando não há dado) e critérios de prioridade explícitos na matriz de criticidade (D07 do inventário), com catálogo distribuído de forma equilibrada entre as praças.

### LGPD — dado pessoal
<!-- - Dados de empregados, imagens de ocupantes, placas capturadas por OCR entrarem na base ou no prompt. -->
  - DESCRIÇÃO: Dados privados e pessoais que não podem entrar na base ou serem processados via prompt. Placa de carro, imagens de ocupantes, dados de empregados e afins.
  - IMPACTO: Acarretar em processos ou sanções da ANPD.
- MITIGAÇÃO: Declarar o que nunca entra na base (inventario.md, seção 5: dados de empregados, placas, imagens de ocupantes), não processar esses dados no prompt e não enviar dados para nenhum local externo.
- APLICAÇÃO: É aplicado na regra 15 do systemprompt.

### Confidencialidade operacional
<!-- - Histórico de falhas e arrecadação indo para API pública de terceiros. -->
- DESCRIÇÃO: Dados de operação indo para API pública de terceiros.
- IMPACTO: Possibilita que os dados sejam vazados.
- MITIGAÇÃO: Não usar API's públicas.
- APLICAÇÃO: Toda a arquitetura, incluindo ambos LLM's, o Llama_guard e o Qwen, pode ser operada localmente, sem uso de API's públicas.

### Excesso de confiança
<!-- - Técnico trata hipótese do modelo como diagnóstico confirmado. -->
- DESCRIÇÃO: Chatbot trata suas hipóteses como fato e não mostra a diferença entre dados factuais e sua inferência.
- IMPACTO: Informação importante é omitida, piorando a conclusão que o operador pode tomar, possivelmente gastando recursos onde não era necessário.
- MITIGAÇÃO: Deve-se exigir ao modelo mostrar todas as informações e não fazer nenhuma decisão sozinho, apenas usar, e mostrar os dados reais e rotular suas respostas com FATO, HIPÓTESE e SUGESTÃO.
- APLICAÇÃO: É aplicado nas regras 4, 7 e 17 do systemprompt

### Contexto estourado
<!-- - Conversa longa ultrapassa a janela e o assistente perde o TAG ou mistura ativos. -->
- DESCRIÇÃO: Uma conversa longa demais faz que dados criticos saiam da janela de contexto
- IMPACTO: Facilita que o modelo alucine, ou que peça por mais informações por que perdeu o contexto necessário. Faz que uma linha de raciocinio entre operador e chat seja interrompida e reinciada, gastando tokens.
- MITIGAÇÃO: Sempre manter os dados criticos dentro da janela, por política de histórico, ou reafirmação desses dados a cada mensagem
- APLICAÇÃO: É aplicado na regra 16 do systemprompt e no caso C10 do evalset
