Sim. Houve uma mudança importante nos últimos meses. O mercado deixou de falar em OCR e passou a falar em Document VLMs (Vision-Language Models) ou OCR-VLMs. A diferença é enorme:

* OCR tradicional: “quais caracteres existem na imagem?”
* OCR-VLM: “o que existe neste documento, qual a estrutura, tabelas, gráficos, fórmulas, leitura correta e significado?”

Para o ACO/Horizon/OR isso é particularmente interessante porque esses modelos já devolvem Markdown estruturado, JSON ou respostas diretamente utilizáveis por um agente.

O que realmente está acontecendo

Os três ecossistemas que estão acelerando mais são:

Empresa	Tendência	Estado atual
Google	Gemini 2.5 Flash/Pro Vision	extremamente forte para scene text e reasoning visual
China	Qwen, PaddleOCR, DeepSeek OCR, GLM OCR	enorme quantidade de modelos open-source
Baidu	PaddleOCR-VL	provavelmente o projeto open mais importante hoje

⸻

1. PaddleOCR-VL 1.6 (Baidu)

Na minha opinião, é o projeto mais importante para acompanhar.

O que mudou

Não é mais apenas OCR.

Ele consegue:

* documentos
* PDFs
* tabelas
* gráficos
* equações
* assinaturas
* carimbos
* formulários
* leitura correta de colunas
* saída em Markdown

A versão 1.6, publicada em junho, elevou ainda mais a precisão com refinamento direcionado das regiões difíceis do documento e pós-treinamento progressivo, alcançando um novo estado da arte no benchmark OmniDocBench.  

O mais interessante:

* apenas 0.9B parâmetros
* roda relativamente barato
* 100+ idiomas
* excelente para RAG

⸻

2. Qwen3-VL

Alibaba está investindo pesadamente.

A filosofia deles é diferente.

Não querem criar um OCR.

Querem um modelo que:

* leia
* entenda
* raciocine
* responda perguntas

sobre imagens.

Na prática:

Imagem
↓
Qwen
↓
"Qual é o número do contrato?"
↓
resposta

sem pipeline intermediário.

Para agentes isso reduz muito a complexidade.

⸻

3. DeepSeek OCR

Outro projeto chinês interessante.

O diferencial é a conversão muito eficiente de documentos para Markdown estruturado, com foco em alimentar pipelines de IA. Em rankings recentes para uso local ele aparece entre as melhores opções para digitalização de PDFs e RAG.  

⸻

4. GLM-OCR

Surpresa de 2026.

Tem apenas cerca de 900 milhões de parâmetros.

Mesmo assim:

* excelente em tabelas
* excelente em fórmulas
* muito rápido

Muita gente começou a adotá-lo porque roda em hardware pequeno.

⸻

5. Google Gemini 2.5 Flash Vision

O Google parece ter seguido outro caminho.

Eles praticamente abandonaram a ideia de “OCR”.

O modelo simplesmente olha a imagem.

Exemplos:

Leia a receita médica.

ou

Extraia todos os medicamentos.

ou

Explique este fluxograma.

Tudo na mesma inferência.

Em comparações recentes, Gemini Flash aparece entre os líderes em OCR de texto em cenas e documentos, especialmente quando combinado com entendimento visual.  

⸻

6. Qianfan-OCR (Baidu)

Esse apareceu recentemente e chamou bastante atenção.

A ideia é eliminar completamente o pipeline clássico.

Imagem →

Markdown

sem:

* detector
* recognizer
* parser

Tudo é um único modelo.

O conceito novo é chamado Layout-as-Thought.

O modelo primeiro “pensa” na estrutura do documento (ordem de leitura, caixas, tipos de elementos) e só então produz a resposta, melhorando documentos complexos.  

Na minha opinião, esse conceito influencia diretamente o futuro dos agentes.

⸻

O conceito novo: OCR virou “Document Intelligence”

Há uns três anos um pipeline típico era:

Imagem
↓
OCR
↓
Texto
↓
LLM

Hoje está ficando assim:

Imagem
↓
Vision Language Model
↓
Markdown
↓
JSON
↓
Agente

ou simplesmente:

Imagem
↓
Vision Model
↓
Resposta

⸻

O que eu estudaria primeiro

Se eu estivesse evoluindo o ACO, minha ordem seria:

1. PaddleOCR-VL 1.6 (referência open-source para documentos)
2. Qwen3-VL (melhor equilíbrio entre OCR e raciocínio)
3. Gemini 2.5 Flash Vision (estado da arte comercial)
4. Qianfan-OCR (arquitetura de próxima geração)
5. GLM-OCR (modelo leve e muito eficiente)
6. DeepSeek-OCR (forte para conversão em Markdown)

⸻

O que mais me interessa para o ACO

Na verdade, eu acho que existe uma oportunidade muito alinhada com a arquitetura do ACO.

Hoje quase todo mundo faz:

Imagem
↓
OCR
↓
LLM

Mas o Horizon poderia fazer algo como:

Imagem
↓
Document Skill
      │
      ├── detecta tipo
      ├── escolhe OCR
      ├── escolhe modelo
      ├── valida qualidade
      ├── extrai metadados
      ├── cria memória
      ├── indexa no Nexus
      ├── calcula confiança
      └── registra observabilidade no Hydra

Ou seja, o OCR deixaria de ser uma ferramenta isolada e passaria a ser uma skill cognitiva dentro do ecossistema do ACO, integrada ao roteamento (OR/DIR), memória (Nexus), observabilidade (Hydra) e orquestração (Horizon). Esse tipo de composição é exatamente a direção para a qual os modelos multimodais estão evoluindo.

Você também pode acompanhar esse tema por meio de um lembrete periódico.
