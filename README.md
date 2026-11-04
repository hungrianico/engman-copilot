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

## 4. Escopo do assistente [voltar aqui depois de fazer o sysprompt]
### 4.1 O que faz:
- Consome o Zabbix e os alarmes gerados por ele
- Recomenda possiveis ações (ex: acionar manutenção, manter em observação, ou sem ação para falsos alarmes ou casos de baixa prioridade)
- Dependendo da informação do Zabbix, informa o caso dos ativos conforme solicitação
### 4.2 O que recusa:
- Inventar dados
- Tratar de dados pessoais
- Oferecer auxilio sobre ativos que não sejam as cameras
**Frase de recusa:** "Eu não posso atender a esta solicitação pois ela foge do meu escopo de atuação"

## 5. Arquitetura da solução
[Tabela com no mínimo 2 modelos e 2 vector stores, critérios objetivos e o trade-off aceito.]

## 6. Base de conhecimento
[Resumo do inventário e como será usada no RAG.]

## 7. System prompt
[Resumo do inventário e como será usada no RAG.]

## 8. Eval set
[Resumo dos 10 casos por categoria + link para o JSON]

## 9. Tools especificadas
[Tabela nome / o que faz / parâmetros + link para tools_spec.json.]

## 10. Riscos e mitigação
[Resumo da matriz: risco ? impacto ? mitigação no prompt ou na arquitetura.]

## 11. Limitações conhecidas [isso nao é obrigatorio]
[Onde o assistente ainda falha ou depende de dado que a Motiva precisaria fornecer.]

## 12. Referências
[Briefing da Motiva, materiais da disciplina, normas citadas (NR-10, NR-35), guias de prompting.]



