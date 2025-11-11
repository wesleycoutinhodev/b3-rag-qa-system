# B3 Q\&A com RAG — Sistema de Perguntas e Respostas Aumentadas (RAG)

## 🎯 Visão Geral do Projeto

Este projeto implementa um sistema de Perguntas e Respostas (Q\&A) focado em documentos e informações da B3 e notícias da área economica, e de empresas da B3 (Bolsa de Valores do Brasil). Utiliza a arquitetura **Retrieval-Augmented Generation (RAG)** para combinar a capacidade de recuperação de informações com o poder de raciocínio de um Large Language Model (LLM).

O fluxo de trabalho inclui a ingestão de um *corpus* de documentos, o processamento de textos em *chunks*, a geração de **Embeddings**, o armazenamento em um índice vetorial (FAISS) e, por fim, a utilização do LLM para gerar respostas contextualmente relevantes.

## 🧠 Detalhe do Uso de Inteligência Artificial (IA)

A IA é a espinha dorsal deste projeto, sendo aplicada em duas etapas principais através da arquitetura RAG:

1.  **Retrieval (Recuperação):** A pergunta do usuário é convertida em um vetor (Embedding). Este vetor é usado para buscar os trechos de texto (*chunks*) mais relevantes no índice FAISS.
2.  **Generation (Geração):** Os trechos de texto recuperados são anexados à pergunta original, formando um **prompt contextualizado**. Este prompt é enviado ao Large Language Model (LLM) para que ele formule uma resposta coerente e baseada estritamente nas informações fornecidas pelo contexto.

O modelo de LLM utilizado para a geração é o **Qwen2.5-7B-Instruct** (`Qwen/Qwen2.5-7B-Instruct`), um modelo otimizado para instruções.

## 💻 Uso de Hugging Face

A plataforma Hugging Face é essencial, pois fornece os modelos de Machine Learning (ML) abertos e de alto desempenho utilizados no projeto:

  * **Large Language Model (LLM):** `Qwen/Qwen2.5-7B-Instruct`. É o modelo responsável por ler o contexto recuperado e gerar a resposta final ao usuário.
  * **Modelo de Embedding:** `BAAI/bge-m3`. Este modelo multilíngue e multifuncional é usado para converter textos em vetores.
  * **Modelo de Re-ranker (Opcional):** `BAAI/bge-reranker-v2-m3`. Usado para refinar a qualidade dos documentos recuperados, reordenando os *chunks* mais relevantes antes de enviá-los ao LLM, melhorando a precisão da resposta.

## 📊 O Papel dos Embeddings (Vetores Semânticos)

**Embeddings** são representações numéricas (vetores) de textos, onde a similaridade de significado entre os textos se traduz na proximidade matemática dos seus vetores correspondentes.

1.  **Criação do Conhecimento:** Cada bloco de texto (chunk) do *corpus* da B3 é transformado em um vetor (embedding) usando o modelo `BAAI/bge-m3`.
2.  **Indexação (FAISS):** Esses vetores são armazenados e indexados no banco de dados vetorial **FAISS** (`faiss-cpu`). O FAISS permite a busca eficiente (em milissegundos) dos vetores que estão semanticamente mais próximos do vetor da pergunta do usuário.
3.  **Configurações:** O processo de criação de embeddings utiliza o parâmetro `CHUNK_TOKENS = 800` para segmentar o texto em blocos de tamanho ideal e `CHUNK_OVERLAP = 100` para garantir a continuidade do contexto entre os blocos.

## 🛠️ Configuração e Instalação

### 1\. Dependências

Instale as bibliotecas necessárias para o processamento de texto, criação de embeddings, busca vetorial e execução do LLM:

```bash
!pip -q install transformers accelerate bitsandbytes sentence-transformers faiss-cpu unstructured pdfminer.six pypdf tiktoken tqdm gdown
```

### 2\. Parâmetros-chave

Os principais parâmetros de configuração do sistema estão definidos no notebook:

| Parâmetro | Valor Padrão | Descrição |
| :--- | :--- | :--- |
| **`MODEL_LLM`** | `Qwen/Qwen2.5-7B-Instruct` | O Large Language Model utilizado para geração.|
| **`EMB_MODEL`** | `BAAI/bge-m3` | O modelo da Hugging Face para gerar Embeddings. |
| **`RERANK_MODEL`** | `BAAI/bge-reranker-v2-m3` | O modelo para reordenar os documentos recuperados. |
| **`CHUNK_TOKENS`** | `800` | Tamanho máximo de tokens por bloco de texto. |
| **`TOPN`** | `5` | Quantidade de *chunks* (contexto) final que é enviado ao LLM. |
| **`RESTORE_FROM_DRIVE`** | `True` | Se **True**, pula o processamento de chunking/embeddings e baixa os artefatos prontos do Google Drive. |

### 3\. Execução

O projeto é executado sequencialmente através das células do notebook:

1.  **Instalação de Dependências:** Garante que todas as bibliotecas estão instaladas.
2.  **Configurações e Diretórios:** Define modelos e parâmetros.
3.  **Persistência via Google Drive:** Permite restaurar artefatos pré-processados (`chunks.jsonl`, `faiss.index`, `meta.jsonl`) para acelerar a inicialização (se `RESTORE_FROM_DRIVE=True`).
4.  **Chunking (Se necessário):** Divide os documentos do *corpus* (`corpus.jsonl`) em pedaços menores (`chunks.jsonl`).
5.  **Embeddings + FAISS (Se necessário):** Cria os vetores de cada *chunk* e constrói o índice FAISS para busca vetorial.
6.  **Busca e Reranker:** Carrega os modelos e o índice, e define as funções de busca.
7.  **LLM Inference:** Carrega o modelo de geração e a função final (`answer`) para responder às perguntas usando o contexto recuperado.
