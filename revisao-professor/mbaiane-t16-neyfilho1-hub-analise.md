# Análise — Ney Penalva Filho (`neyfilho1-hub`)

**Projeto:** O Fim da Evasão Inesperada de Talentos — People Analytics: Previsão de Turnover Voluntário com MLP
**Repositório:** [neyfilho1-hub/FGV-aulas](https://github.com/neyfilho1-hub/FGV-aulas)
**Arquivos analisados:** `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`, `mbaiane-t16-neyfilho1-hub-O-Fim-da-Evasao-Inesperada-de-Talentos_NOVO.pptx`

---

## 1. Abertura

Ney, parabéns por concluir o projeto! Com 23 anos à frente da área de RH e hoje como Diretor na Rede Ápice, faz todo sentido você ter escolhido turnover voluntário como tema — é o problema que você mais sente na pele, e a apresentação mostra isso: você chegou a um framework de custo de reposição (1,5x a 2,0x o salário anual) e de retorno projetado (R$ 1,2M/ano) com uma clareza de negócio que boa parte da turma não teve. Esse é justamente o ponto forte do trabalho — o desafio agora é fazer o notebook sustentar tecnicamente os números que a apresentação promete.

## 2. Resumo do projeto

O projeto usa uma base sintética de 1.200 colaboradores (gerada por fórmula no próprio notebook, não dados reais da Rede Ápice) para prever turnover voluntário. Compara duas arquiteturas de MLP entre si — V1 (16 neurônios) e V2 (32→16 neurônios, com regularização L2 e early stopping) — sem comparação com nenhum modelo de ML tradicional, e a apresentação propõe um caso de negócio com ROI estimado (R$ 150-200k de custo de construção vs. R$ 1,2M/ano de economia, payback de 2-3 meses).

## 3. Nota por critério

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 8,3/10**
1.1. Métrica de sucesso nomeada como receita/custo: **5** — pptx, slide 2 (cartão "Impacto Financeiro"): "Cada saída estratégica custa ~R$ 60 mil em rescisão, recrutamento e perda de produtividade" e slide final "R$ 1,2M Economia Anual — Em custos evitados de rescisão e recrutamento".
1.2. Métrica quantificada: **5** — R$ 60 mil, R$ 1,2M, 25% de redução de turnover, 15% mais retenções, todos numéricos no pptx.
1.3. Conexão entre métrica técnica e impacto de negócio: **3** — o slide 2 (cartão "Modelo validado") liga Recall 83%/AUC 0,884 a "captura 8 em cada 10 colaboradores em risco real de saída", mas não há uma derivação explícita (quantos colaboradores identificados × R$ economizado = R$ 1,2M); é uma conexão narrativa, não uma conta mostrada. Além disso, como detalhado em "Correção técnica" abaixo, os números de Recall/AUC citados não são reproduzíveis a partir do notebook como commitado, o que fragiliza essa conexão.

**2. Viabilidade econômica (ROI) — 7,5/10**
2.1. Custo de construção estimado: **5** — pptx: "R$ 150–200k em infraestrutura, ciência de dados e integração".
2.2. Custo de sustentação estimado: **2** — não há valor de custo recorrente (retraining, monitoramento, infra de produção); o único elemento de pós-deploy é a frase "Shadow Mode → Teste A/B → produção", sem custo associado.
2.3. Retorno esperado com número: **5** — "R$ 1,2M Economia Anual" explícito.
2.4. Comparação custo vs. retorno: **4** — pptx: "2 a 3 meses, com base na economia projetada de R$ 1,2M/ano" — payback explícito e coerente em ordem de grandeza, mas o intervalo exato do pptx não bate com a conta implícita nos próprios dados do aluno: R$ 150-200k / (R$ 1,2M/12) ≈ 1,5-2 meses (o intervalo do pptx é 2-3 meses; só o valor "2" é comum aos dois). É uma inconsistência do próprio pptx que a versão anterior desta análise tratou como coerência sem ressalva.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 0,0/10**
3.1. Discute alternativa de automação/regra determinística: **1** — não há, em notebook ou pptx, qualquer menção a um critério simples (ex.: "sinalizar quem está há mais de 3 anos sem promoção e faz horas extras") como alternativa antes de partir para rede neural.

**4. ML tradicional vs. Redes Neurais — 0,0/10**
4.1. Compara explicitamente contra ML tradicional: **1** — busquei no notebook inteiro por "Regress", "logistic", "baseline" (fora do rótulo da própria MLP V1) — nenhuma ocorrência. O notebook original do curso para este mesmo tema (`12-people-analytics-turnover.ipynb`, antes da reescrita) comparava MLP vs. Regressão Logística; a versão do aluno removeu essa comparação por completo.
4.2. Baseline simples de fato executado e comparado: **1** — a Célula 7 compara duas MLPs entre si (V1 "Baseline" de 16 neurônios vs. V2 de 32-16), não um modelo de ML tradicional interpretável vs. rede neural.

**5. Aderência ao conteúdo do curso — 10,0/10**
5.1. Nomeia arquitetura vista em aula: **5** — MLP com ativação ReLU e otimizador Adam, explicitado na Célula 6 (markdown) e Célula 7 (código) — conteúdo de Aulas 1-3.
5.2. Arquitetura adequada ao tipo de dado: **5** — dados tabulares (idade, salário, satisfação, distância) — MLP é escolha correta.
5.3. *(não aplicável — problema não é de texto)*

**6. Aderência ao template de projeto — 3,75/10**
6.1. Cobre os 7 blocos do `templates_projetos_ia.md`: **4** — todos os 7 blocos têm alguma evidência (Visão Geral e ROI muito bem cobertos no pptx; Split de dados correto na Célula 5; Métricas/Testes estruturados na Célula 9-13), mas dois blocos são rasos: "Coleta e Preparação de Dados" é 100% dado sintético gerado por fórmula (Célula 3), sem fonte real da Rede Ápice nem discussão de qualidade/missing values; e "MLOps: Deploy e Monitoramento" se resume a uma frase no último slide do pptx ("Shadow Mode → Teste A/B → produção" + menção a LGPD), sem estratégia de monitoramento de drift ou cadência de retraining.
6.2. Profundidade do bloco 7 (MLOps): **1** — conferi o notebook completo submetido no PR (`aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`, PR #2 de `neyfilho1-hub/FGV-aulas`) e ele não tem nenhuma célula sobre deploy/monitoramento. A única menção existe no pptx, slide 3: "Shadow Mode → Teste A/B → produção" e "Projeto em conformidade com a LGPD. Nenhuma decisão automatizada sem validação humana." É uma sequência de rollout pré-produção, não um plano de monitoramento — não há o quê monitorar em produção (nenhuma métrica nomeada), nenhuma frequência de retraining e nenhum critério de quando o modelo deve ser revisado ou aposentado.

**7. Correção técnica — 6,9/10**
7.1. Código executa do início ao fim sem erro: **4** — reexecutei o notebook integralmente (`jupyter nbconvert --execute`) e ele roda sem lançar exceções. Porém, como submetido no GitHub, **todas** as 8 células de código têm `execution_count: null` e zero outputs — o aluno nunca comitou uma execução real, então não há evidência própria de que ele validou o resultado antes de reportar números na apresentação.
7.2. Split antes de pré-processamento: **5** — Célula 5: `train_test_split` ocorre antes de `scaler.fit_transform(X_train)`, sem vazamento.
7.3. Métrica de avaliação adequada à distribuição de classes: **1** — ao reexecutar a Célula 3, descobri que o limiar `prob_turnover > 0.48` **nunca é atingido**: calculei a fórmula de `score_risco` manualmente com `seed=42` e o `prob_turnover` máximo entre os 1.200 registros é 0,2088 (média 0,00989). Resultado: `Turnover` = 0 para 100% da base. Ao rodar a Célula 9 com esse dado, a tabela de métricas real fica: Acurácia V1 = 100,0% / V2 = 48,0%, Precisão = Recall = F1 = 0,0% (nenhum caso positivo previsto), e **ROC-AUC = NaN** (indefinido com uma única classe em `y_test`). Isso não bate com os números do pptx ("Recall 83% • ROC-AUC 0,884 • Acurácia 82%").
7.4. Baseline avaliado no mesmo split que o modelo principal: **5** — V1 e V2 usam os mesmos `X_train_scaled`/`X_test_scaled` (Célula 7), ainda que "baseline" aqui seja outra MLP, não um modelo tradicional.

**8. Qualidade do código — 5,8/10**
8.1. Seeds fixadas: **5** — `np.random.seed(42)` (Célula 3) e `random_state=42` em `train_test_split` e nos dois `MLPClassifier`.
8.2. Dependências declaradas: **1** — ausente. Os demais notebooks do repositório (inclusive a versão original deste mesmo arquivo) têm uma célula `%pip install -q numpy pandas scikit-learn matplotlib tensorflow` logo no início; a reescrita do aluno começa direto com `import pandas as pd` (Célula 1), sem declarar dependências.
8.3. Organização em funções/seções: **4** — bem seccionado com markdown explicativo por etapa, mas é código linear célula-a-célula, sem funções reutilizáveis.

**9. Honestidade dos resultados — 0,0/10**
9.1. Múltiplas seeds/execuções ou justificativa: **1** — apenas `seed=42`, uma única execução, sem qualquer menção à necessidade de checar robustez.
9.2. Seção de limitações presente: **1** — ausente tanto no notebook quanto no pptx. Pelo contrário, o pptx afirma "Modelo validado" com três métricas de alta confiança, sem qualificar que a base é 100% sintética (não dados reais da Rede Ápice) nem que os resultados vêm de uma única execução — e, como mostrado acima, esses números nem sequer são reproduzíveis a partir do código commitado.

## 4. Pontos fortes

- **2.1/2.3 Custo de construção e retorno esperado** (5/5): o pptx traz um caso de negócio de ROI com custo de construção (R$ 150-200k) e retorno anual (R$ 1,2M) explícitos e numéricos — algo raro na turma.
- **2.4 Comparação custo vs. retorno** (4/5): o payback citado (2-3 meses) é coerente em ordem de grandeza com a conta implícita nos próprios números do pptx (R$ 150-200k / (R$ 1,2M/12) ≈ 1,5-2 meses), mas o intervalo exato não bate — inconsistência do pptx do aluno que não deve ser lida como coerência plena.
- **7.2 Split antes de pré-processamento** (5/5): `train_test_split` ocorre antes de `scaler.fit_transform(X_train)` (Célula 5), sem vazamento dos dados de teste no ajuste do `StandardScaler`.
- **5.1/5.2 Arquitetura nomeada e adequada ao tipo de dado** (5/5): MLP com ReLU/Adam corretamente identificada e justificada para dados tabulares (Célula 6-7), com duas versões (16 neurônios; 32→16 com regularização L2 e early stopping) e uso de Permutation Feature Importance para interpretabilidade (Célula 13).
- **8.1 Seeds fixadas** (5/5): `np.random.seed(42)` (Célula 3) e `random_state=42` em `train_test_split` e nos dois `MLPClassifier` (Célula 5 e 7).

## 5. Pontos de melhoria

- **1.3 Conexão entre métrica técnica e impacto de negócio** (3/5): o pptx liga Recall 83%/AUC 0,884 a "captura 8 em cada 10 colaboradores em risco real de saída" de forma narrativa, sem mostrar a conta (quantos colaboradores identificados × R$ economizado = R$ 1,2M) — e essa conexão fica ainda mais frágil porque, como mostra 7.3, esses números não são reproduzíveis a partir do notebook commitado.
- **2.2 Custo de sustentação estimado** (2/5): não há nenhum valor de custo recorrente de retraining/monitoramento/infra de produção; o único elemento pós-deploy no pptx é "Shadow Mode → Teste A/B → produção", sem custo associado — é o item de ROI que mais penaliza a nota de negócio.
- **7.3 Métrica de avaliação adequada à distribuição de classes** (1/5): bug crítico no threshold da variável sintética (Célula 3) — `prob_turnover > 0.48` nunca é atingido (máximo real ≈ 0,2088) — zera a taxa de turnover e torna Recall/AUC indefinidos (NaN) quando o notebook é de fato executado, invalidando os números "Recall 83% • ROC-AUC 0,884" citados no pptx.
- **4.1/4.2 Comparação com ML tradicional e baseline executado** (1/5): nenhuma comparação com Regressão Logística ou outro modelo interpretável — o notebook original do curso para este tema fazia exatamente essa comparação; a reescrita do aluno a removeu e compara apenas duas MLPs entre si.

## 6. Nota final

**5,0 / 10** — A apresentação de negócio (ROI, custo, payback) está entre as mais completas da turma e puxa a nota para cima, mas o núcleo técnico do trabalho está quebrado: a variável-alvo sintética tem um bug de limiar que zera a taxa de turnover, tornando Recall/AUC indefinidos quando o notebook é de fato executado — números que não sustentam os indicadores citados na apresentação —, e falta qualquer comparação com ML tradicional ou seção de limitações.

**Nível de maturidade: PoC/protótipo.** O pptx traz custo, retorno e payback explícitos (2.1/2.3/2.4), o que à primeira vista sugere um piloto já planejado, mas esses números repousam sobre métricas de modelo (Recall 83%, AUC 0,884) que não são reproduzíveis a partir do notebook commitado (ver 7.3), e o bloco de MLOps (6.2) se resume a uma sequência de rollout ("Shadow Mode → Teste A/B → produção") sem qualquer métrica de monitoramento, frequência de retraining ou critério de revisão/aposentadoria do modelo — falta validar o modelo tecnicamente antes mesmo de cogitar um piloto controlado.

## 7. Task list para evoluir o trabalho

**1. Aderência ao negócio**
- [ ] **1.3 Conexão entre métrica técnica e impacto de negócio (nota 3/5):** o Bloco D é direto: "Métrica-Alvo de Negócio: A métrica que traduz o resultado estatístico em dinheiro ou eficiência operacional." Hoje o pptx afirma que o modelo "captura 8 em cada 10 colaboradores em risco real de saída", mas não mostra a conta — explicite `economia = TP (verdadeiros positivos no teste) × R$ 60 mil (custo médio por saída)` e mostre esse número batendo com o R$ 1,2M projetado. Isso também exige primeiro corrigir o bug de 7.3, já que hoje o modelo não produz nenhum TP real. — ver slide 2 (cartão "Modelo validado") do pptx e Célula 9

**2. Viabilidade econômica (ROI)**
- [ ] **2.2 Custo de sustentação estimado (nota 2/5):** o Bloco B contrasta automação com IA numa linha de tabela de 3 colunas — "Manutenção" | "Atualiza-se a regra manualmente" | "Retreina-se o modelo periodicamente". Esse é exatamente o custo recorrente que falta estimar no pptx: frequência de retraining, custo de time de dados para monitorar drift, e infraestrutura de scoring mensal para os "5.000+ colaboradores" citados no slide 2. — ver slide "Próximo Passo" do pptx e item 6.2 abaixo

**3. Necessidade real de IA**
- [ ] **3.1 Discute alternativa de automação/regra determinística (nota 1/5):** o Bloco B traz o teste direto, em três nós separados de fluxograma: "A lógica pode virar regras fixas (se-então)?", "O problema muda muito, ou tem grande variação?" e "Há dados históricos suficientes para aprender um padrão?" — três "sim" seguidos = provavelmente um projeto de IA; se alguma resposta for "não", automação simples resolve com menos custo e mais previsibilidade. Antes de justificar a MLP, mostre no próprio notebook o resultado de uma regra simples (ex.: "sinalizar quem está há mais de 3 anos sem promoção e faz horas extras") e compare seu recall/precisão contra a rede neural. — nenhuma menção encontrada no notebook nem no pptx

**4. ML tradicional vs. Redes Neurais**
- [ ] **4.1 Compara explicitamente contra ML tradicional (nota 1/5):** o próprio notebook de referência do curso para este tema, `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb` original (antes da reescrita do aluno), fazia essa comparação explícita — MLP vs. Regressão Logística. A reescrita removeu essa discussão por completo; reintroduza ao menos um parágrafo (markdown) justificando por que a MLP foi escolhida frente a um modelo linear/interpretável. — ver Célula 6 (markdown) e o notebook original do curso
- [ ] **4.2 Baseline simples de fato executado e comparado (nota 1/5):** o Bloco C é direto: "Regra prática: comece sempre por um baseline simples." A Célula 7 hoje compara duas MLPs entre si (V1 de 16 neurônios "Baseline" vs. V2 de 32-16), o que não substitui um baseline de ML tradicional. Treine um `LogisticRegression` em `X_train_scaled`/`y_train` e adicione seus resultados à mesma tabela da Célula 9, ao lado de V1/V2. — ver Célula 7 e Célula 9

**6. Aderência ao template de projeto**
- [ ] **6.1 Cobertura dos 7 blocos do template (nota 4/5):** dois blocos ficam rasos — "Coleta e Preparação de Dados" é 100% sintético gerado por fórmula (Célula 3), sem qualquer dado real da Rede Ápice nem discussão de qualidade. Adicione uma nota explicando por que dados reais não foram usados (LGPD, disponibilidade, prazo) e como o modelo seria recalibrado quando dados reais existirem. O outro bloco raso é o de MLOps, detalhado no item 6.2 abaixo. — ver Célula 3 e slide "Próximo Passo" do pptx
- [ ] **6.2 Profundidade do bloco 7/MLOps (nota 1/5):** hoje o único conteúdo é o slide 3 do pptx — "Shadow Mode → Teste A/B → produção" — uma sequência de rollout, não um plano de monitoramento. Adicione: o quê monitorar em produção (ex.: distribuição de `prob_turnover`, taxa de turnover real mês a mês, drift nas variáveis de entrada), com que frequência (ex.: revisão mensal, já que o slide 2 fala em avaliar "todos os 5.000+ colaboradores mensalmente"), e um critério numérico de quando retreinar ou aposentar o modelo (ex.: "retreinar se o AUC observado cair X pontos abaixo do AUC de validação"). — ver slide "Próximo Passo" do pptx

**7. Correção técnica**
- [ ] **7.1 Código executa sem erro (nota 4/5):** reexecute o notebook do início ao fim e salve os outputs antes de submeter — como está no GitHub, todas as 8 células de código têm `execution_count: null`, então não há evidência própria de que o resultado foi validado antes de reportar os números no pptx. — ver todas as células de código do notebook
- [ ] **7.3 Métrica de avaliação adequada à distribuição de classes (nota 1/5):** corrija o bug de threshold da variável sintética `Turnover` (Célula 3) — `prob_turnover > 0.48` nunca é atingido (máximo real ≈ 0,2088 com seed=42), zerando a taxa de turnover e tornando `roc_auc_score` indefinido (NaN) com uma única classe em `y_test`. Reexecute a Célula 9 após corrigir e reconcilie os números resultantes com os citados no pptx (Recall 83%, AUC 0,884, Acurácia 82%). — ver Célula 3 e Célula 9

**8. Qualidade do código**
- [ ] **8.2 Dependências declaradas (nota 1/5):** isso não é conteúdo do curso propriamente (é prática de reprodutibilidade de ambiente, não de IA de negócio), mas os demais notebooks do repositório — inclusive a versão original deste mesmo arquivo — têm uma célula `%pip install -q numpy pandas scikit-learn matplotlib tensorflow` logo no início; adicione essa célula antes do `import pandas as pd` (Célula 1).
- [ ] **8.3 Organização em funções/seções (nota 4/5):** também não é conteúdo do curso (é engenharia de software, não IA de negócio), mas vale como boa prática: extrair o treino/avaliação das duas MLPs (Célula 7 e 9) em uma função reutilizável (`treinar_e_avaliar(config)`), facilitando testar outras arquiteturas ou adicionar o baseline de Regressão Logística (item 4.2) sem duplicar código.

**9. Honestidade dos resultados**
- [ ] **9.1 Múltiplas seeds/execuções (nota 1/5):** o Bloco A é direto — "Cada comparação foi rodada com 3 sementes aleatórias diferentes (não uma vez só), para separar ganho real de sorte da rodada." — e o próprio notebook 12 original (o mesmo arquivo em que este aluno trabalhou) reforça isso na Célula 1: "Robustez confirmada, não só sorte de uma seed. A seção de validação com múltiplas seeds, mais adiante, mostra que a vantagem da regressão logística sobre a rede neural (MLP) não é um acidente da seed 42." Depois de corrigir o bug de 7.3, repita o treino de V1/V2 com pelo menos 3 seeds (ex.: 42, 7, 123) e reporte média ± desvio padrão de Recall/AUC. — ver Célula 3 e Célula 7
- [ ] **9.2 Seção de limitações presente (nota 1/5):** não encontrei no material do curso uma citação específica sobre "seção de limitações" — é mais próximo do espírito geral do Bloco A de não inflar resultados sem checar robustez do que um sub-item ensinado explicitamente. O pptx afirma "Modelo validado" com três métricas de alta confiança sem qualificar que a base é 100% sintética (não dados reais da Rede Ápice) nem que os resultados vêm de uma única execução. Adicione uma seção (célula final do notebook ou slide do pptx) citando essas limitações e o risco de viés em variáveis demográficas (idade, distância). — ver Célula 0 do notebook e slide 2 (cartão "Modelo validado") do pptx

## 8. Tópicos para o aluno revisar

- **Validação da variável-alvo e checagem de distribuição de classes** (Aula 2 / Bloco C — Design de Projetos de IA) — motivado por `12-people-analytics-turnover.ipynb` (Célula 3): o limiar de 0.48 sobre `prob_turnover` nunca é atingido pela distribuição gerada, produzindo 0% de casos positivos; vale revisar a prática de sempre inspecionar `y.value_counts()` logo após gerar ou carregar os dados, antes de treinar qualquer modelo.
- **Métricas de avaliação sob classe única/desbalanceamento extremo** (Aula 2-3, seção de métricas) — motivado pela Célula 9: `roc_auc_score` retorna `NaN` quando `y_test` tem uma única classe; o notebook reporta a métrica sem esse caso-limite ser checado.
- **ML tradicional vs. Redes Neurais como comparação obrigatória** (Bloco B — Redes Neurais / notebook original `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb` do curso) — motivado pela ausência de Regressão Logística no notebook reescrito: o material do curso usa exatamente esse contraste no notebook 12 original; vale revisar por que a rede neural se justifica frente a um modelo mais simples e interpretável.
- **MLOps: monitoramento de drift e custo de sustentação** (Bloco D / `templates_projetos_ia.md`, bloco 7) — motivado pelo slide final do pptx ("Shadow Mode → Teste A/B → produção"): falta detalhar como o drift será detectado e qual o custo recorrente de manter o modelo em produção.
- **Honestidade dos resultados e reexecução antes de reportar números** (Bloco C — Design de Projetos de IA) — motivado pela discrepância entre os números do pptx e os resultados reproduzidos a partir do notebook commitado (todas as células com `execution_count: null`): vale revisar a prática de sempre reexecutar o notebook do zero e salvar os outputs antes de citar métricas em uma apresentação.
