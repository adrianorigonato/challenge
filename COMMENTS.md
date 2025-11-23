# COMMENTS

## 1. Decisão da arquitetura utilizada

O projeto foi estruturado em uma arquitetura cliente–servidor simples, mas bem separada por responsabilidades:

- **Backend em Python + FastAPI**  
  - Exposição de uma API REST com rotas para:
    - ingestão de arquivos (`/api/ingest`);
    - criação e continuidade de conversas (`/api/conversation/start` e `/api/conversation/chat`);
    - análise da conversa e geração de conteúdos personalizados (`/api/conversation/{id}/analyze-and-generate`).
  - A lógica de negócio foi dividida em módulos: extração de conteúdo, chunking + embeddings, motor de chat RAG, análise pedagógica da conversa e geração de conteúdos personalizados.

- **RAG com PostgreSQL + pgvector**  
  - O conteúdo dos arquivos (PDF, áudio, vídeo, imagem) é extraído/transcrito, dividido em *chunks* e armazenado em uma tabela `documents` com um vetor de embedding.
  - A busca de contexto para o chat é feita via similaridade vetorial (pgvector), permitindo respostas baseadas nos documentos ingeridos.
  - Há tabelas separadas para:
    - histórico de conversa (`conversation`);
    - análises pedagógicas (`profile_information`);
    - conteúdos personalizados gerados (`personalized_learning_contents`).

- **Modelos externos (LLM / embeddings / visão / áudio)**  
  - Embeddings são gerados usando um modelo via OpenRouter.
  - As respostas do chat, a análise pedagógica da conversa e os roteiros personalizados são gerados usando modelos da Groq.
  - Áudio, vídeo e imagens usam serviços de transcrição e visão da Groq para transformar mídia em texto antes da indexação.

- **Frontend simples em HTML + JS + TailwindCSS**  
  - SPA bem leve em JavaScript puro, consumindo a API via `fetch`.
  - Três abas principais:
    - **Chat**: onde o usuário conversa com o assistente.
    - **Estudar**: onde são exibidos apenas os conteúdos personalizados gerados.
    - **Ingestão de arquivos**: para enviar PDFs/áudios/vídeos/imagens para a base.
  - O estado de conversa (ID da conversa, formato preferido, cache da análise) é mantido no lado do cliente em variáveis globais de `main.js`.

Essa combinação foi escolhida por ser simples de entender, fácil de evoluir e adequada para um protótipo de RAG focado em experiência de aprendizado personalizada.

---

## 2. Lista de bibliotecas de terceiros utilizadas

**Backend (Python)**

- `fastapi` – framework web para a API HTTP.
- `pydantic` – modelos de dados e validação de payloads.
- `psycopg2` – driver para conexão com PostgreSQL.
- `requests` – chamadas HTTP para serviços externos (Groq, OpenRouter).
- `pdfplumber` – extração de texto de arquivos PDF.
- Extensão `vector` do PostgreSQL (pgvector) – armazenamento e busca de embeddings vetoriais.

**Frontend**

- **Tailwind CSS** (via CDN) – estilização rápida e responsiva da interface (botões, cards, layout, etc).

**Serviços externos (APIs)**

- **OpenRouter** – geração de embeddings e modelo de chat específico para geração de conteúdos personalizados.
- **Groq** – modelos de:
  - chat para o assistente conversacional e análise pedagógica;
  - transcrição de áudio/vídeo (Whisper-like);
  - visão para descrição e extração de texto de imagens.

---

## 3. O que eu melhoraria se tivesse mais tempo

Se houvesse mais tempo, alguns pontos que eu evoluiria seriam:

1. **Experiência do usuário (UX) e interface**
   - Exibir de forma mais clara a análise de subtemas e níveis (básico/intermediário/avançado/domina) antes de mostrar os conteúdos.
   - Permitir o download/compartilhamento dos roteiros gerados em formatos como PDF ou Markdown.
   - Mostrar indicadores de carregamento/erro mais ricos (por exemplo, skeletons, ícones, toasts).

2. **Camada de arquitetura e escalabilidade**
   - Tornar as chamadas pesadas (ingestão, transcrição, análise+geração) assíncronas, com filas ou *jobs* em background.
   - Organizar melhor os módulos em pastas (por exemplo, `services/`, `repositories/`, `routers/`) para deixar mais claro o domínio.

3. **Qualidade de código e observabilidade**
   - Adicionar testes automatizados (unitários e de integração) para as principais funções (chunking, ingestão, busca vetorial, análise).
   - Melhorar o tratamento de erros e logs, incluindo correlação de requisições e métricas básicas (tempo de resposta, número de conteúdos gerados etc.).

4. **Busca e relevância**
   - Ajustar parâmetros de chunking (tamanho, overlap) e experimentar filtros por metadados (tipo de documento, título, fonte).
   - Incluir um campo de busca na aba “Estudar” para filtrar conteúdos por subtema ou tipo (vídeo/áudio/texto).

5. **Segurança e multiusuário**
   - Implementar autenticação e associação das conversas/ingestões a um usuário específico.
   - Limitar tamanho de upload e validar tipos de arquivo de maneira mais robusta.

---

## 4. Quais requisitos obrigatórios não foram entregues

Até o momento desta entrega, **todos os requisitos obrigatórios principais propostos para o desafio foram contemplados**, incluindo:

- Ingestão de arquivos (PDF, áudio, vídeo, imagem) com extração/transcrição e armazenamento em base vetorial.
- Chat baseado em RAG utilizando o conteúdo dos documentos ingeridos.
- Análise da conversa para identificação de subtemas e níveis de conhecimento do usuário.
- Geração de conteúdos personalizados para estudo, exibidos em uma aba separada no frontend.
- Interface web simples permitindo conversar, ingerir arquivos e visualizar os conteúdos gerados.

Caso exista algum requisito obrigatório adicional no enunciado específico da avaliação (por exemplo, autenticação obrigatória, painel administrativo, testes automatizados com cobertura mínima, ou deploy em ambiente X), esse ponto pode ser considerado como trabalho futuro e não está contemplado nesta versão.
