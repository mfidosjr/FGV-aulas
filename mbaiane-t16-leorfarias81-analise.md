# Análise — Leonardo Ribeiro de Farias (`leorfarias81`)

**Projeto:** Visão Computacional no Varejo — Detecção de Prateleira Vazia (v2)
**Repositório:** [leorfarias81/FGV-aulas](https://github.com/leorfarias81/FGV-aulas)
**Arquivos analisados:** `14-deteccao-prateleira-vazia-v2.ipynb`, `14-apresentacao-executiva-prateleira-vazia.pptx` (idêntica à versão enviada localmente, `mbaiane-t16-leorfarias81-Deteccao-Prateleira-Vazia.pptx`)
**Não avaliado:** `Aula 1`, `Aula 2`, `Modulo2_1_...`, `aplicacoes-de-negocio/` (material clonado do curso) e `MBAIANEG-M_RNAN_Bloco-D_20260820.pdf` (material do curso, não é plano de projeto do aluno) — nenhum desses é autoria do aluno.

---

## 1. Abertura

Leonardo, parabéns por concluir o projeto! Com 20 anos de experiência e hoje como Gerente de Tecnologia na Terex Latin America, chama a atenção que você não tenha resolvido o problema mais óbvio (menos código, resultado bonito) e sim o mais honesto: construiu uma segunda versão do notebook justamente para corrigir, com method e dados, as fraquezas que uma primeira tentativa tinha exposto. Vale registrar que o problema escolhido — visão computacional para prateleira vazia em varejo — é o mesmo do notebook de referência `05-visao-computacional-varejo.ipynb` do curso, não um problema original da Terex (fabricante de equipamentos, não varejo); ainda assim, o aprofundamento metodológico que você aplicou em cima dele é o mais rigoroso entre os trabalhos revisados até aqui.

## 2. Resumo do projeto

O projeto simula imagens de gôndola (64×64, 8 perfis de câmera, oclusão e ruído realistas) para classificar cada seção como "cheia" ou "precisa de reposição", comparando três níveis de complexidade: um limiar sobre o brilho médio (baseline determinístico), um Gradient Boosting sobre 9 features espaciais artesanais, e uma CNN com data augmentation, dropout e early stopping. A avaliação usa split por câmera (treino em câmeras 0-4, teste em câmeras nunca vistas 6-7), métricas além de acurácia (precisão, recall, F1, PR-AUC, matriz de confusão), calibração do limiar de decisão por custo de negócio e validação com 3 seeds — concluindo que a CNN vence os outros dois degraus de forma consistente e estável.

## 3. Nota por critério

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 8,3/10**
1.1. Métrica de sucesso nomeada como receita ou custo: **3** — a meta primária do piloto é operacional ("-40% no tempo médio de ruptura", slide "Impacto Esperado"), não nomeada literalmente como R$ ou receita; a conexão com venda perdida é feita de forma narrativa no slide 1 do pptx ("Cada hora de ruptura não detectada é venda que vai embora para o concorrente") e ecoada na célula 0, que discute a mesma ideia com outras palavras (venda perdida e migração do cliente para o concorrente), mas o KPI em si continua sendo tempo, não dinheiro.
1.2. Métrica quantificada: **5** — -40% (ruptura), -25% (horas de varredura manual) e 15% (teto de alarme falso) estão todos explícitos no pptx (slide "Impacto Esperado e Caminho até a Entrega").
1.3. Conexão explícita entre métrica técnica e impacto de negócio: **5** — é o ponto mais forte do trabalho: a célula 24 define recall como "das rupturas que realmente aconteceram, quantas o modelo detectou? [...] recall baixo = venda perdida"; a célula 27-28 assume `custo_FN = 5 × custo_FP`, recalibra o limiar e traduz a matriz de confusão em "rupturas perdidas" e "alertas a toa" com números concretos (FN: 11→0, FP: 4→47); e a célula 36 confronta esse resultado "ótimo" contra o teto de 15% de alarme falso do próprio pitch, mostrando que ele viola a restrição operacional. Poucos trabalhos da turma amarram métrica técnica e negócio com esse nível de detalhe.

**2. Viabilidade econômica (ROI) — 1,25/10**
2.1. Custo de construção estimado: **1** — ausente em notebook e pptx; nenhuma menção a custo de câmeras, infraestrutura de treino/inferência ou horas de desenvolvimento.
2.2. Custo de sustentação estimado: **1** — ausente; a célula 37 e o slide "Validação Segura" descrevem *o que* será monitorado (drift, taxa de alertas confirmados), mas não *quanto custa* monitorar/retreinar.
2.3. Retorno esperado estimado com número: **3** — há números (-40%, -25%), mas são metas operacionais de piloto, nunca convertidas em receita evitada ou R$.
2.4. Comparação explícita custo vs. retorno: **1** — nenhum payback, ROI% ou breakeven em nenhum dos dois materiais.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 10/10**
3.1. Discute explicitamente a alternativa de regra determinística e por que foi (ou não) descartada: **5** — é o melhor exemplo do critério entre os trabalhos revisados. A célula 2 estrutura o próprio notebook como uma escada: "Limiar sobre o brilho médio — *o problema se resolve só com 'quanta luz tem na foto'?* É o que uma loja conseguiria implementar sem modelo nenhum. Zero parâmetros, interpretável, roda em microssegundos [...] Se o degrau 3 não bater o degrau 1, a resposta certa para o negócio é ficar no degrau 1." A regra simples não é descartada por suposição — é implementada, medida (célula 15, acurácia 0,894 no teste) e só perde porque o degrau 3 prova ser melhor com dados.

**4. ML tradicional vs. Redes Neurais — 10/10**
4.1. Compara explicitamente contra um modelo de ML tradicional: **5** — `HistGradientBoostingClassifier` (célula 17) como degrau intermediário entre a regra e a CNN, com justificativa de features (célula 16) e leitura honesta de importância por permutação (células 18-19).
4.2. Baseline simples de fato executado e comparado no notebook: **5** — os três níveis são executados no mesmo split de teste e reunidos na tabela da célula 25 (acurácia 0,894 / 0,922 / 0,957).

**5. Aderência ao conteúdo do curso — 10/10**
5.1. Nomeia arquitetura vista em aula, entre as Aulas 1-6: **5** — CNN com `Conv2D`, `MaxPooling2D`, `GlobalAveragePooling2D`, `Dropout` e `RandomFlip/RandomTranslation/RandomBrightness/RandomContrast` como *augmentation* (célula 21), conteúdo da Aula 4.
5.2. Arquitetura adequada ao tipo de dado: **5** — dado é imagem (gôndola 64×64), CNN é a escolha correta, com `EarlyStopping` monitorando a câmera de validação nunca vista (célula 22).
5.3. *(não aplicável — problema não é de texto)*

**6. Aderência ao template de projeto — 7,5/10**
6.1. Cobre os 7 blocos do `templates_projetos_ia.md`: **5** — os 7 blocos aparecem com evidência real, ainda que distribuídos entre notebook e pptx em vez de um único documento formal: (1) Visão geral — célula 0 + slide "O Desafio Atual"; (2) Coleta de dados — célula 6 (simulação) + slide "Nossos Dados" (fontes reais: câmera, planograma, ERP, PDV); (3) Split — células 12-13 (split por câmera, o bloco mais bem executado do trabalho); (4) Seleção de algoritmos — células 2, 14, 16, 20 (limiar → boosting → CNN, com justificativa); (5) Treinamento/otimização — células 20-22 (loss `binary_crossentropy`, dropout, early stopping, augmentation — sem busca sistemática de hiperparâmetros, ver Pontos de melhoria); (6) Testes e métricas — células 24-35 (precisão, recall, F1, PR-AUC, matriz de confusão, calibração por custo, multi-seed — o bloco mais completo entre os trabalhos revisados); (7) MLOps — célula 37 (tabela explícita "Aqui vs. Em produção": inferência na borda, monitoramento de data/concept drift) + slide "Validação Segura" (shadow mode, piloto em 3-5 lojas, expansão regional).
6.2. Profundidade do bloco 7 (MLOps): **3** — a célula 37 nomeia indicadores concretos de monitoramento (*data drift* de brilho/contraste por câmera, *concept drift* por mudança de planograma, taxa de alertas confirmados) e o slide "Validação Segura" descreve uma estratégia de implantação em estágios (shadow mode → piloto em 3-5 lojas → expansão regional, com a regra simples como modo degradado) — mais concreto que a maioria dos trabalhos da turma. Mas nenhum dos dois materiais define a frequência de monitoramento/retreino nem um critério numérico de quando o modelo deve ser revisado ou aposentado: a única menção a cadência é "recalibrado periodicamente, com a razão de custo definida pelo negócio" (célula 37), sem periodicidade explícita nem limiar de gatilho.

**7. Correção técnica — 10/10**
7.1. Código executa do início ao fim sem erro: **5** — `execution_count` sequencial de 1 a 21 em todas as células de código, sem saída de erro.
7.2. Split antes de qualquer pré-processamento que aprende parâmetros: **5** — o limiar do degrau 1 é escolhido varrendo apenas `brilho[m_tr]`/`y[m_tr]` (célula 15); a extração de features (célula 17) é uma transformação determinística sem `fit`, então não há vazamento; o `RandomFlip`/`RandomBrightness` da CNN roda só em treino (comportamento padrão do Keras).
7.3. Métrica de avaliação adequada à distribuição/problema: **5** — mesmo com classes praticamente balanceadas (50,1% vs. 49,9%, célula 9), o aluno não se contenta com acurácia: reporta precisão, recall, F1 e PR-AUC (célula 25) e ainda quebra a avaliação no subconjunto ambíguo, que é onde a decisão de negócio realmente se joga.
7.4. Baseline avaliado no mesmo split/dados que o modelo principal: **5** — os três modelos são avaliados exatamente no mesmo `m_te` (células 15, 17, 22, reunidos na célula 25).

**8. Qualidade do código — 10/10**
8.1. Seeds fixadas para reprodutibilidade: **5** — `SEED = 42` fixado em `numpy`/`tensorflow`/`rng` (célula 5), e o experimento é repetido com 3 seeds diferentes (célula 33) — vai além do mínimo pedido pelo critério.
8.2. Dependências declaradas: **5** — `%pip install` explícito na célula 4.
8.3. Organização em funções/seções, sem duplicação excessiva: **5** — funções reutilizáveis (`perfil_camera`, `gerar_imagem_gondola`, `gerar_dataset`, `brilho_medio`, `extrair_features`, `construir_cnn`, `avaliar`, `limiar_por_custo`) que são chamadas tanto na análise de seed única quanto no laço de 3 seeds, evitando repetir lógica.

**9. Honestidade dos resultados — 10/10**
9.1. Resultado reportado com mais de 1 execução/seed: **5** — célula 33: pipeline completo (geração de dados, threshold, boosting, CNN) repetido para 3 seeds com rotação de câmeras de teste, reportando média ± desvio-padrão (célula 34) e discutindo o porquê da diferença de estabilidade entre baseline e CNN (célula 36).
9.2. Seção de limitações presente: **5** — célula 1 ("Limitações deste notebook": imagens simuladas, rótulo derivado da simulação, acurácias não são desempenho esperado em produção) e uma segunda ressalva na célula 36 sobre a calibração de custo violar o teto de alarme falso do pitch — duas seções de limitações, não uma.

## 4. Pontos fortes

- **1.3 Conexão explícita entre métrica técnica e impacto de negócio** (5/5): a célula 24 define recall como "das rupturas que realmente aconteceram, quantas o modelo detectou? [...] recall baixo = venda perdida"; a célula 27-28 assume `custo_FN = 5 × custo_FP`, recalibra o limiar (FN: 11→0, FP: 4→47) e a célula 36 confronta esse resultado "ótimo" contra o teto de 15% de alarme falso do próprio pitch — poucos trabalhos da turma amarram métrica técnica e negócio com esse nível de detalhe.
- **3.1 Discute explicitamente a alternativa de regra determinística e por que foi (ou não) descartada** (5/5): a célula 2 estrutura o notebook como uma "escada de complexidade" (limiar → boosting → CNN), implementando e medindo a regra simples (célula 15, acurácia 0,894) antes de justificar cada degrau seguinte — o melhor exemplo do critério entre os trabalhos revisados.
- **6.1 Cobre os 7 blocos do `templates_projetos_ia.md`** (5/5): o split por câmera (células 12-13, treino em câmeras 0-4, teste em câmeras nunca vistas 6-7) é "o bloco mais bem executado do trabalho", simulando o cenário real de deploy em vez de um split aleatório.
- **9.1 Resultado reportado com mais de 1 execução/seed** (5/5): a célula 33 repete o pipeline completo para 3 seeds com rotação de câmeras de teste, reportando média ± desvio-padrão (célula 34) e explicando *por que* a conclusão se inverteu entre a v1 (baseline vencia) e a v2 (CNN vence e é 7× mais estável).

## 5. Pontos de melhoria

- **2.1/2.2 Custo de construção e de sustentação estimados** (1/5 cada): nenhuma menção a custo de câmeras, infraestrutura de treino/inferência, horas de desenvolvimento ou custo de monitoramento/retreino, em notebook ou pptx — o critério de maior peso da rubrica fica sem qualquer estimativa.
- **2.3 Retorno esperado estimado com número** (3/5): as metas -40% (ruptura) e -25% (varredura manual) estão quantificadas, mas nunca convertidas em receita evitada ou custo evitado em R$ — ficam como percentuais operacionais de piloto.
- **2.4 Comparação explícita custo vs. retorno** (1/5): nenhum payback, ROI% ou breakeven aparece em notebook nem em pptx, impedindo qualquer decisão de investimento baseada em número.

## 6. Nota final

**6,4 / 10** — Tecnicamente é o trabalho mais rigoroso entre os revisados (correção técnica, qualidade de código e honestidade dos resultados no máximo, com validação multi-seed e split por câmera que poucos trabalhos profissionais fazem), e a conexão entre métrica técnica e impacto de negócio é a mais bem construída da turma. Mas a nota é limitada pelo critério de maior peso da rubrica: não há nenhuma estimativa de custo de construção, sustentação ou retorno financeiro — a viabilidade econômica do projeto simplesmente não foi endereçada.

**Nível de maturidade: PoC/protótipo.** A validação técnica (multi-seed, split por câmera, calibração de custo) já tem rigor de piloto, mas sem nenhuma estimativa de custo de construção/sustentação (2.1, 2.2) ou retorno em R$ (2.3), e sem frequência de retreino ou limiar de revisão do modelo (6.2), ainda falta a base econômica para decidir uma entrada em piloto controlado — o próximo passo é quantificar esses números antes de levar a solução além do notebook.

## 7. Task list para evoluir o trabalho

**1. Aderência ao negócio**
- [ ] **1.1 Métrica de sucesso nomeada como receita ou custo (3/5):** o Bloco B é direto — "Qual métrica de negócio define sucesso? Não acurácia técnica — receita, custo evitado, tempo economizado, risco reduzido." Suas metas (-40% de ruptura, -25% de varredura) ainda são operacionais, não R$. O notebook `03-previsao-demanda.ipynb` (célula 23) resolve exatamente esse gap no mesmo domínio de varejo/estoque: "a métrica MAE deve sempre ser traduzida em impacto financeiro (custo de estoque parado vs. custo de ruptura) antes de decidir qual modelo colocar em produção." Aplique a mesma lógica: estime quanto custa uma hora de ruptura não detectada (ticket médio × frequência × margem) e quanto custam as horas de varredura manual evitadas (custo/hora × horas economizadas). — ver célula 0 e slide "Impacto Esperado" do pptx

**2. Viabilidade econômica (ROI)**
- [ ] **2.1/2.2 Custo de construção e de sustentação (1/5 cada):** o Bloco C trata isso como "custo computacional — treino e inferência têm custo, meça contra o orçamento disponível", e o próprio Bloco C descreve a manutenção como parte do ciclo ("1 Deploy... 2 Monitorar... 3 Detectar drift... 4 Retreinar... repete o ciclo"), não um evento único. Estime: custo de câmeras/infra adicional (se houver), horas de desenvolvimento, custo de treino/inferência em produção, e custo recorrente de monitoramento/retreino. — ver célula 37 (tabela "Do notebook para o projeto") e o slide "Impacto Esperado e Caminho até a Entrega" do pptx
- [ ] **2.3 Retorno esperado estimado com número (3/5):** o Bloco D chama isso de "Métrica-Alvo de Negócio: A métrica que traduz o resultado estatístico em dinheiro ou eficiência operacional", e recomenda usar placeholders (`[Inserir % de economia]`) quando falta o dado exato em vez de deixar sem tradução nenhuma. O notebook `03-previsao-demanda.ipynb` (célula 23) ilustra esse mesmo raciocínio de tradução no domínio de estoque/varejo. Formalize -40%/-25% como `[Inserir ticket médio] × frequência de ruptura evitada` e `[Inserir custo/hora] × horas de varredura economizadas`, deixando a suposição explícita. — ver slide "Impacto Esperado e Caminho até a Entrega" do pptx
- [ ] **2.4 Comparação explícita custo vs. retorno (1/5):** o Bloco C fecha o raciocínio de seleção de algoritmo com "custo computacional — treino e inferência têm custo, meça contra o orçamento disponível", mas o curso não formaliza uma fórmula de payback ou ROI% específica. Depois de estimar 2.1/2.2 (custo) e 2.3 (retorno), aplique `ROI = (retorno - custo) / custo` ou um payback em meses para justificar (ou não) o investimento na CNN frente à regra simples de brilho médio. — ausente em notebook e pptx

**6. Aderência ao template de projeto**
- [ ] **6.2 Profundidade do bloco 7 (MLOps) (3/5):** o Bloco C descreve o ciclo completo de MLOps — "1 Deploy... 2 Monitorar... 3 Detectar drift... 4 Retreinar... repete o ciclo, com dados e hiperparâmetros ajustados" — e você já cobre bem os três primeiros passos (célula 37: data/concept drift por câmera; slide "Validação Segura": shadow mode, piloto em 3-5 lojas). Falta fechar o quarto passo com números: defina a frequência de retreino (ex: mensal, ou a cada N% de queda no F1 do subconjunto ambíguo) e um limiar explícito de quando a razão de custo (hoje fixa em 5×) deve ser revisada ou o modelo aposentado em favor da regra simples de fallback. — ver célula 37 e o slide "Validação Segura" do pptx

*(Critérios 3, 4, 5, 7, 8 e 9 tiraram nota máxima em todos os sub-itens — sem pendências de melhoria.)*

Além disso, vale resolver o exercício 2 proposto no fim do notebook (célula 39) — variar a razão custo_FN/custo_FP de 1 a 20 e apresentar a curva de trade-off recall × alertas/dia à diretoria, já que a célula 36 identificou que a razão atual (5×) viola o teto de 15% de alarme falso do pitch — e considerar rodar uma busca de hiperparâmetros (grid ou random search) sobre a CNN (nº de filtros, taxa de dropout, learning rate), já cobertos no tópico de revisão da seção 8.

## 8. Tópicos para o aluno revisar

- **Viabilidade econômica e ROI de projetos de IA** (Bloco C — Design de Projetos de IA, seção MLOps do `templates_projetos_ia.md`) — motivado pela ausência total de custo de construção/sustentação e de qualquer comparação com retorno em `14-deteccao-prateleira-vazia-v2.ipynb` e no pptx: o template pede explicitamente essa comparação, e ela não aparece em nenhum dos dois materiais.
- **Otimização de hiperparâmetros** (Aula 3 — Treinando Redes Neurais com Keras e PyTorch) — motivado pela célula 21 (`construir_cnn`): a arquitetura (número de filtros, camadas, dropout) foi escolhida uma vez e usada em todas as seeds, sem comparação sistemática com alternativas.
- **Tradução de métricas técnicas em métricas financeiras** (Bloco C — Design de Projetos de IA) — motivado pelo slide "Impacto Esperado" do pptx: as metas de -40%/-25% já estão quantificadas, o passo que falta é convertê-las em R$ para fechar o círculo entre AUC/recall e o resultado que a diretoria de fato acompanha.
