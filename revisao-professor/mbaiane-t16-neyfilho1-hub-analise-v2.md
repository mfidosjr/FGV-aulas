# Análise v2 — Ney Penalva Filho (`neyfilho1-hub`)

**Projeto:** O Fim da Evasão Inesperada de Talentos — People Analytics: Previsão de Turnover Voluntário com MLP
**Repositório:** [neyfilho1-hub/FGV-aulas](https://github.com/neyfilho1-hub/FGV-aulas) (PR #2 em `mfidosjr/FGV-aulas`)
**Arquivo analisado (v2, commit `1759a9a`):** `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`
**Nota v1:** 5,0/10 → **Nota v2: 6,0/10**

---

## 1. Abertura

Ney, obrigado por reenviar — confirmei que desta vez o push chegou de verdade ao repositório, e o bug crítico específico que eu tinha apontado (o limiar de 0,48 nunca sendo atingido) foi corrigido na arquitetura: a nova geração de dados separa a variável de risco (usada só para gerar o rótulo) do modelo treinado (que aprende com as features reais), o que é uma estrutura tecnicamente mais correta. Você também endereçou, na forma, todos os outros pontos que eu tinha pedido — baseline heurística, comparação com ML tradicional, ROI detalhado, MLOps. Isso é progresso real e reconheço o esforço.

Só que, ao reexecutar o notebook do zero (de novo, porque ele chegou sem nenhuma evidência de execução, exatamente como na v1), encontrei que o problema de fundo não desapareceu — só migrou de lugar, e desta vez atinge o coração do trabalho: a rede neural, que deveria ser a protagonista de um projeto de "Redes Neurais Aplicadas a Negócios", tem desempenho pior que aleatório, e o caso de negócio inteiro (ROI, payback) foi construído sobre um número de Recall que nunca bateu com o que o código realmente produz. Preciso que você trate isso com a mesma prioridade que tratou o bug anterior.

## 2. O que mudou desde a v1 (verificado com reexecução completa, do zero)

Reexecutei o notebook inteiro em ambiente limpo — como ele chegou sem nenhum output salvo (`execution_count: null` em todas as 9 células, de novo), essa foi a única forma de saber o que o código de fato produz.

| Item da v1 | Situação na v1 | O que você fez na v2 | Verificado |
|---|---|---|---|
| **7.3** Threshold 0,48 nunca atingido (bug crítico) | `prob_turnover` máximo real ≈0,2088, zerava o turnover, AUC=NaN | Nova arquitetura: `score_risco` com intercepto calibrado, `prob_turnover = sigmoid(score_risco)`, rótulo gerado por `prob_turnover > 0.50` | O bug específico não se repete — mas um novo problema de calibração aparece no lugar (ver seção 3). |
| **3.1** Sem alternativa de regra determinística | Ausente | Regra heurística (`Anos_Sem_Promocao≥3 & Horas_Extras==1`), avaliada no mesmo conjunto | Executada sem vazamento; recall real 29,4% (teste), você reportou "~35%" no comentário da PR — divergência pequena, plausivelmente imprecisão. |
| **4.1/4.2** Sem comparação com ML tradicional | Só duas MLPs comparadas entre si | Regressão Logística e Random Forest agora comparados no mesmo split que a MLP | Executado e confirmado sem vazamento (`StandardScaler` ajustado só no treino). |
| **6.2** MLOps sem conteúdo | Só uma frase no pptx | PSI, cadência de retreino (trimestral ou PSI≥0,20), limiar de Recall (<75%) | Ver ressalva grave na seção 3 — o PSI implementado não testa drift de verdade. |
| **9.2** Sem seção de limitações | Ausente | Ainda ausente | Não foi adicionada nesta v2. |
| **9.1** Sem múltiplas seeds | Ausente | Ainda ausente (só `seed=42`) | Não foi adicionado nesta v2, apesar de já pedido na v1. |

## 3. Os dois achados centrais desta v2

### 3.1 A rede neural tem desempenho pior que aleatório — e isso não é mencionado em lugar nenhum

Reexecutei a célula de comparação (célula 9) e obtive esta tabela real:

| Modelo | Acurácia | Precisão | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Regra Determinística | 89,0% | 19,2% | 29,4% | 23,3% | — |
| Regressão Logística | 90,7% | 36,6% | **88,2%** | 51,7% | **0,974** |
| Random Forest | 95,3% | 66,7% | 35,3% | 46,2% | 0,925 |
| **Rede Neural MLP** | 50,0% | 6,0% | 52,9% | 10,7% | **0,534** |

A MLP — o modelo que dá nome à disciplina e ao seu projeto — tem AUC de 0,534, praticamente empatado com uma moeda jogada ao ar, muito atrás da Regressão Logística (0,974). A causa técnica: o `early_stopping` do `MLPClassifier` monitora acurácia de validação por padrão, e como só 5,8% da base é da classe positiva, "prever tudo negativo" já dá ~94% de acurácia — o treino para em 17 iterações sem aprender sinal nenhum. Diferente de `LogisticRegression`/`RandomForestClassifier`, o `MLPClassifier` do scikit-learn não aceita o parâmetro `class_weight='balanced'`, então o desbalanceamento nunca é corrigido para a rede neural especificamente.

Isso não é, em si, um problema de honestidade — é um problema técnico real, e até seria um achado honesto e valioso se reconhecido (é exatamente o tipo de resultado que outro colega da turma reportou com transparência: "o baseline simples supera a rede neural"). O problema é que **isso não é mencionado em nenhum lugar do notebook**: a seção de ROI (célula 14) segue tratando a MLP como o modelo vencedor, com "Recall = 83%", um número que não aparece em nenhuma saída de código.

### 3.2 O caso de negócio inteiro repousa sobre dois números que não sobrevivem à execução real

| | Alegado (célula 14, texto) | Real (execução da célula 3/9) |
|---|---|---|
| Taxa de turnover/ano | 21% (252 casos) | **5,8% (69 casos)** |
| Custo total do turnover/ano | R$ 15,12M | **R$ 4,14M** |
| Recall da MLP | 83% | **52,9%** |
| Colaboradores identificados/ano | 209 | **~36,5** |
| Retidos (10% de conversão) | 20 | **~3,65** |
| Economia anual | R$ 1,2M | **~R$ 219 mil** |
| Economia líquida/mês | R$ 95.000 | **~R$ 13.250** |
| **Payback** | **1,84 meses** | **~13,2 meses** |

A cadeia de contas do texto está aritmeticamente correta dados os números que você usou — o problema é que nenhum desses dois números de entrada (taxa de turnover, Recall da MLP) bate com o que o próprio código produz quando executado. A causa da primeira divergência: o intercepto do `score_risco` foi calibrado olhando a **média** de `prob_turnover` (≈0,166), não a fração da população que de fato cruza o limiar de 0,50 usado para definir o rótulo — a sigmoide faz com que só a cauda superior (~5-6%) ultrapasse esse ponto, não a média inteira.

O payback real (~13,2 meses) está fora da faixa "1,8 a 2 meses" citada como validação do caso de negócio — é aproximadamente 7 vezes mais otimista do que a execução real sustenta.

### 3.3 O monitoramento de drift (PSI) não testa nada de fato

A célula 16 compara `y_prob_mlp` contra `model_mlp.predict_proba(X_test_scaled.copy())` — ou seja, os mesmos dados de teste, sem nenhuma perturbação, contra si mesmos. O PSI reportado é 0,0000 exato, o que é matematicamente garantido por essa construção, independentemente da qualidade do modelo ou de qualquer mudança real nos dados. A fórmula do PSI está correta, mas a simulação não demonstra capacidade de detectar drift genuíno — é o mesmo padrão de "monitoramento fake" já visto em outro projeto da turma.

## 4. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 7,5/10** *(v1: 8,3/10)*
1.3. Conexão entre métrica técnica e impacto de negócio: **2** *(v1: 3)* — agora há uma derivação explícita (célula 14, TP×R$60k), o que é a forma certa de fazer essa conexão; mas os números de entrada dessa conta (Recall e taxa de turnover) são os fabricados/não verificados da seção 3.2, então a nota cai apesar do avanço de forma.

**2. Viabilidade econômica (ROI) — 5,0/10** *(v1: 7,5/10)*
2.2. Custo de sustentação estimado: **4** *(v1: 2)* — R$60.000/ano agora presente e razoável.
2.3. Retorno esperado com número: **2** *(v1: 5)* — R$1,2M alegado vs. ~R$219 mil real — divergência de 5,5x.
2.4. Comparação custo vs. retorno: **1** *(v1: 4)* — payback de 1,84 meses alegado vs. ~13,2 meses real — divergência de ~7x, fora de qualquer faixa razoável de arredondamento.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 7,5/10** *(v1: 0,0/10)*
3.1. Discute alternativa de regra determinística: **4** *(v1: 1)* — implementada e comparada de fato; não é 5 porque essa comparação está dentro de uma tabela cujo resultado central (MLP) está comprometido pelos achados da seção 3.

**4. ML tradicional vs. Redes Neurais — 10,0/10** *(v1: 0,0/10)*
4.1/4.2. Comparação explícita e baseline executado: **5/5** *(v1: 1/1)* — Regressão Logística e Random Forest genuinamente implementados e comparados no mesmo split. Nota à parte, não pontuada aqui: essa comparação revelou que a rede neural é o pior dos três modelos (seção 3.1) — um achado tecnicamente valioso que deveria estar no centro da discussão do trabalho, não omitido.

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 6,25/10** *(v1: 3,75/10)*
6.2. Profundidade do bloco 7 (MLOps): **3** *(v1: 1)* — cadência e limiares numéricos agora definidos, mas o teste de PSI (seção 3.3) não valida detecção de drift de fato.

**7. Correção técnica — 6,9/10** *(v1: 6,9/10, sem mudança líquida)*
7.1. Código executa sem erro: **4** *(mantido)* — roda do início ao fim quando executado por mim, mas chegou sem nenhuma evidência própria de execução, de novo.
7.3. Métrica adequada à distribuição: **1** *(mantido)* — o bug específico da v1 foi corrigido, mas um novo erro de calibração no mesmo ponto do pipeline (geração do rótulo) produz o mesmo tipo de consequência: um número central do projeto (taxa de turnover) que não bate com a execução real.

**8. Qualidade do código — 5,8/10** *(mantido)* — dependências ainda não declaradas (sem célula `%pip install`), mesmo ponto da v1.

**9. Honestidade dos resultados — 0,0/10** *(mantido, v1: 0,0/10 — pelo mesmo motivo estrutural, agora mais grave)*
9.1. Múltiplas seeds/execuções: **1** — ainda não implementado, apesar de já pedido explicitamente na v1.
9.2. Seção de limitações: **1** — ainda ausente; seria exatamente o lugar certo para reconhecer que a MLP performou pior que os baselines e que o ROI depende de premissas não verificadas.

## 5. Nota final

**6,0 / 10** *(v1: 5,0/10)* — Há progresso técnico real: a estrutura de dados agora separa corretamente o processo gerador do rótulo do modelo treinado, a comparação com ML tradicional existe e funciona, e o ROI/MLOps têm a forma certa (mesmo que os números não sustentem). Mas a nota sobe pouco porque o problema de honestidade da v1 não foi resolvido — só migrou: agora ele contamina simultaneamente a premissa do problema (taxa de turnover 3,65x inflada) e o modelo central do trabalho (MLP com AUC=0,534, pior que aleatório, e isso nunca é discutido). O caso de negócio inteiro (payback de 1,84 meses) foi construído sobre esses dois números, e a execução real do seu próprio código mostra que o payback verdadeiro é de aproximadamente 13 meses.

**Nível de maturidade: PoC/protótipo.** A estrutura de MLOps (PSI, cadência, limiares) já tem a forma de um piloto controlado, mas o monitoramento de drift implementado não testa nada de fato (seção 3.3), e o modelo que seria colocado em produção (a MLP) tem desempenho pior que o baseline mais simples do próprio trabalho — nenhuma decisão de negócio deveria ser tomada sobre esses números até que sejam reconciliados com a execução real.

## 6. O que preciso que você corrija para a v3

1. **Prioridade máxima — a MLP:** investigue por que o `MLPClassifier` não aprende (early stopping com `class_weight` ausente e classe minoritária de 5,8%) — considere usar `class_weight` manual via `sample_weight` no `.fit()`, ajustar o critério de early stopping, ou reportar honestamente que a Regressão Logística supera a rede neural neste problema (o que também seria um resultado válido, se reconhecido).
2. **Prioridade máxima — a calibração da taxa de turnover:** ajuste o intercepto de `score_risco` olhando a fração real da população que cruza `prob_turnover > 0.50` (não a média de `prob_turnover`), até obter algo próximo de 21% de fato, e confirme rodando `df['Turnover'].mean()`.
3. **Refaça a seção de ROI (célula 14)** com os números reais que saírem das correções acima — a conta em si está bem estruturada, só precisa dos inputs corretos.
4. **Substitua a simulação de PSI** por uma que de fato perturbe os dados "atuais" (ex: aumentar artificialmente `Horas_Extras` ou deslocar `Salario_Mensal`) antes de comparar com o "esperado" — hoje o PSI=0 é garantido por construção, não uma prova de estabilidade.
5. **Rode o notebook do início ao fim com kernel limpo e salve os outputs antes de comitar** — este é o mesmo ponto pedido na v1 e ainda não atendido; é a causa raiz de todos os achados desta rodada.
6. **Adicione a validação multi-seed e a seção de limitações**, ambas já pedidas na v1 e ainda ausentes.

Quando estiver pronto, me avise que faço uma nova revisão (v3) em cima da correção.
