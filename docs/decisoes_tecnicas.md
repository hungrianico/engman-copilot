### Justificativa de escolhas das Stacks
#### Modelo - Qwen 2.5:7B
Tanto o Qwen 2.5 quanto o Llama 3.1 são modelos que permitem execução local e utilização em aplicações comerciais, respeitando suas respectivas condições de licenciamento.  
Optamos pelo Qwen 2.5:7B por sua licença Apache 2.0, que oferece maior permissividade para distribuição e utilização, além de seu desempenho em pt-BR e uma documentação voltada à implementação e integração em aplicações.  
Entendemos que trocamos uma janela de contexto maior do Llama 3.1 por um desenvolvimento que faça mais sentido com nosso projeto. 
#### Orquestração - Via LangChain
Decidimos usar LangChain para orquestração do workflow pela necessidade de integração entre LLMs, embedding, databases e APIs.  
O uso de uma framework especializada para orquestração proporciona maior modularidade do projeto, nos deixando com uma estrutura de integrações de fácil implementação.
#### Vector Store - ChromaDB
Entre ChromaDB e FAISS escolhemos o ChromaDB, por ser um RAG convencional de fácil configuração.  
Além de ser nativamente construido em Python, tem armazenamento de documentos e metadados implementados nativamente.  
O FAISS seria uma opção melhor em performance, mas consideramos o ChromaDB melhor em funcionalidade.
#### Embeddings - Sentence Transformers
Considerando a natureza do projeto, optamos por usar a framework Sentence Transformers localmente para geração de embeddings. Acreditamos que a segurança e privacidade seja crucial para os dados que estamos trabalhando, fazendo com que o desenvolvimento local dos embeddings seja preferível.  
Ainda que haja a manutenção de um servidor local, sabemos que a privacidade e o controle dos dados fazem valer tal custo de infraestrutura.
#### Interface
