# Handoff de Continuidade: Projeto Lumina

## 1. Estado Atual do Projeto
Entrega final consolidada na branch main do fork jlmaximo/FGV-aulas, integrando o caso de negócio de portabilidade inteligente de crédito para a fincare brasileira Lumina (modelo B2B2C).

## 2. Entregáveis do Projeto
1. Notebook didático de IA:
   - aplicacoes-de-negocio/14-portabilidade-credito-lumina.ipynb
2. Plano de projeto de IA (Markdown):
   - plano-projeto-lumina.md
3. Plano de projeto de IA (PDF oficial):
   - plano-projeto-lumina.pdf
4. Apresentação executiva (3 slides em PPTX):
   - Pare-de-Pagar-Juros-que-Ninguem-Deveria-Pagar.pptx

## 3. Resultados Principais do Experimento
- Validação multi-seed (seeds 42, 7 e 123) com 2.000 contratos simulados:
  - Rede Neural (MLPClassifier): AUC-ROC médio de 0,883 +- 0,010
  - Regressão Logística (Baseline): AUC-ROC médio de 0,861 +- 0,011
- Estabilidade entre sementes:
  - Seed 42: MLP 0,894 vs Regressão Logística 0,857
  - Seed 7: MLP 0,879 vs Regressão Logística 0,852
  - Seed 123: MLP 0,876 vs Regressão Logística 0,873 (empate técnico)
- Decisão de produção:
  - Escolha oficial pela Regressão Logística calibrada via Platt Scaling/Isotônica.
  - Justificativa de negócio: explicabilidade nativa e transparente para o trabalhador CLT, gerando justificativas diretas no aplicativo com baixo custo computacional em arquitetura serverless (Netlify Functions).

## 4. Convenções do Projeto
- Padrão de commits: Conventional Commits (feat, docs, fix, test, chore).
- Idioma: Português brasileiro em toda a documentação e comentários.
- Regras de estilização: Sem uso de em-dashes no texto gerado.
- Integridade: Resultados numéricos do notebook preservados de execução real.
- Isolamento: Documento HANDOFF.md da raiz pertence ao professor e foi preservado intacto.

## 5. Pendências
- Nenhuma pendência técnica ou bloqueante para entrega do MBA FGV.
- Próximos passos de roadmap: coleta de desfechos reais no fluxo do app e integração com Open Finance em fase de escala.
