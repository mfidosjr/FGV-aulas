# Handoff de Continuidade: Projeto Lumina

## 1. Estado Atual do Projeto
Entrega final revisada na branch main do fork jlmaximo/FGV-aulas, incorporando as melhorias de viabilidade econômica (ROI), critérios numéricos de MLOps, refatoração de código (DRY) e seção de limitações recomendadas na revisão.

## 2. Entregáveis do Projeto
1. Notebook didático de IA:
   - aplicacoes-de-negocio/14-portabilidade-credito-lumina.ipynb
2. Plano de projeto de IA (Markdown com Seção 8 de ROI):
   - plano-projeto-lumina.md
3. Plano de projeto de IA (PDF oficial atualizado):
   - plano-projeto-lumina.pdf
4. Apresentação executiva (3 slides em PPTX):
   - Pare-de-Pagar-Juros-que-Ninguem-Deveria-Pagar.pptx

## 3. Resultados Principais e Métricas de Negócio
- Validação multi-seed (seeds 42, 7 e 123) com 2.000 contratos simulados:
  - Rede Neural (MLPClassifier): AUC-ROC médio de 0,883 +- 0,010
  - Regressão Logística (Baseline): AUC-ROC médio de 0,861 +- 0,011
  - Estabilidade: empate técnico na seed 123 (MLP 0,876 vs Regressão Logística 0,873).
- Decisão de produção: Regressão Logística calibrada pela explicabilidade nativa no app.
- Viabilidade econômica (ROI) por empresa cliente (500 colaboradores):
  - Receita anual bruta: R$ 150.000 (R$ 25/colab/mês) | Receita retida (90% retenção): R$ 135.000.
  - Valor devolvido aos colaboradores: R$ 232.000/ano (80 contratos portados x R$ 2.900).
  - Custo no Ano 1: R$ 13.200 (Construção: R$ 6.000 + Sustentação: R$ 7.200).
  - ROI Ano 1: 10,36x (1.036%) | Payback do investimento inicial: 0,48 mês (~15 dias).
- MLOps: Gatilho numérico de retreino (queda > 5 p.p. na taxa de sucesso) e reavaliação trimestral.
- Limitações: Reconhecimento de dados 100% sintéticos e necessidade de feedback loop com dados reais.

## 4. Convenções do Projeto
- Padrão de commits: Conventional Commits (feat, docs, fix, test, refactor).
- Idioma: Português brasileiro em toda a documentação e comentários.
- Regras de estilização: Sem uso de em-dashes no texto gerado.
- Integridade: Resultados numéricos do notebook preservados de execução real.
- Isolamento: Documento HANDOFF.md da raiz pertence ao professor e foi preservado intacto.

## 5. Pendências
- Nenhuma pendência técnica ou bloqueante para entrega do MBA FGV.
- Próximos passos de roadmap: coleta de desfechos reais no fluxo do app e integração com Open Finance em fase de escala.
