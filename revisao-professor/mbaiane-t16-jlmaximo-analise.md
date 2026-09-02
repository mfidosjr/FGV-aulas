# Análise — Jorge Lucas Martins Máximo (`jlmaximo`)

**Projeto:** Lumina — Diagnóstico e Portabilidade Inteligente de Crédito
**Repositório:** [jlmaximo/FGV-aulas-1](https://github.com/jlmaximo/FGV-aulas-1)
**Arquivos analisados:** `aplicacoes-de-negocio/14-portabilidade-credito-lumina.ipynb`, `plano-projeto-lumina.md`, `HANDOFF-lumina.md`, apresentação `Pare-de-Pagar-Juros-que-Ninguem-Deveria-Pagar.pptx`

---

## 1. Abertura

Jorge, parabéns por concluir o projeto! Como Especialista Comercial na Salesforce, com 9 anos de experiência conectando produto e cliente, faz muito sentido você ter escolhido um problema onde o custo de errar não é financeiro para a empresa, mas sim de **confiança e churn** — exatamente o tipo de risco que você lida na prática comercial todos os dias. O cuidado de transformar a saída do modelo em uma frase explicável para o usuário final ("sua taxa está X acima do mercado, a portabilidade economiza R$ Y") mostra instinto de produto que vai além do exercício técnico.

## 2. Resumo do projeto

O projeto Lumina simula 2.000 contratos de crédito (consignado, veículo e imobiliário) de trabalhadores CLT e treina um modelo para prever se uma portabilidade seria vantajosa, comparando uma Regressão Logística (baseline) contra um MLPClassifier (16→8 neurônios, ReLU) usando AUC-ROC. O rótulo é construído com uma interação não linear proposital entre spread de taxa, saldo devedor e prazo restante, e a validação é repetida em três seeds para checar a robustez da vantagem da rede neural — que, apesar de vencer no AUC médio, é descartada em favor da Regressão Logística por causa da explicabilidade exigida pelo produto.

## 3. Nota por critério

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 7,5/10**
1.1. Métrica de sucesso nomeada como receita/custo: **4** — as metas são "taxa de sucesso de portabilidades recomendadas >80%", "churn anual de empresas contratantes <10%" e "engajamento >60%" (`plano-projeto-lumina.md`, seção 1); churn é proxy direto de receita recorrente B2B2C, mas não é nomeado literalmente como R$.
1.2. Métrica quantificada: **5** — os três alvos (80%, 10%, 60%) estão explícitos como hipóteses numéricas na seção 1 do plano; dois deles (80% e 10%) são repetidos no slide 3 do PPTX — o valor de 60% (engajamento) só aparece na seção 1 do plano.
1.3. Conexão entre métrica técnica e impacto de negócio: **3** — a matriz de confusão na seção 6 do plano conecta falso positivo/negativo a "risco de churn B2B (Impacto Crítico)" e a "oportunidade perdida", e a Conclusão (seção 10 do notebook) traduz exemplos individuais em R$ (economia de R$ 4.200 por contrato). Porém a métrica agregada (AUC de 0,86) nunca é traduzida num número de negócio consolidado — falta o passo de "quanto isso vale para a carteira/base de usuários".

**2. Viabilidade econômica (ROI) — 2,5/10**
2.1. Custo de construção estimado: **1** — ausente; não há estimativa de custo de dados, treino ou infraestrutura inicial.
2.2. Custo de sustentação estimado: **2** — a seção 7 do plano descreve o processo ("revisão manual mensal", "retreino orientado a feedback"), mas sem qualquer número de custo associado.
2.3. Retorno esperado com número: **4** — há números concretos: a faixa de receita do modelo B2B2C (R$ 15 a R$ 40/funcionário/mês, seção 1 do notebook) e exemplos individuais de economia (R$ 2.900 e R$ 4.200 por contrato, seções 6 e 10). Falta apenas agregar isso num retorno total esperado do projeto.
2.4. Comparação custo vs. retorno: **1** — nenhum payback, ROI% ou breakeven é calculado; os números de custo e retorno nunca são colocados lado a lado.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 10/10**
3.1. Discute alternativa de automação/regra simples: **5** — o slide 1 do PPTX afirma explicitamente que "ferramentas atuais (calculadoras genéricas, simuladores bancários e análise manual) usam regras estáticas que ignoram a interação entre três fatores", e a seção 2 do notebook (`14-...ipynb`) argumenta que "modelos lineares... têm dificuldade em capturar essas interações tridimensionais sem engenharia manual de atributos" — alternativa simples é discutida e descartada de forma justificada.

**4. ML tradicional vs. Redes Neurais — 10/10**
4.1. Compara explicitamente contra ML tradicional: **5** — Regressão Logística vs. MLP, com justificativa de negócio clara (seção 10, "Conclusão").
4.2. Baseline executado e comparado de fato: **5** — ambos treinados no mesmo split e comparados na tabela da seção 8 ("Comparação lado a lado": AUC 0,857 vs. 0,894).

**5. Aderência ao conteúdo do curso — 10/10**
5.1. Nomeia arquitetura vista em aula: **5** — MLPClassifier com ReLU, duas camadas ocultas (Aula 2/3).
5.2. Arquitetura adequada ao tipo de dado: **5** — dados tabulares contratuais, MLP é escolha correta.
5.3. *(não aplicável — problema não é de texto)*

**6. Aderência ao template de projeto — 7,5/10**
6.1. Cobre os 7 blocos do `templates_projetos_ia.md`: **5** — `plano-projeto-lumina.md` cobre os 7 blocos integralmente e com profundidade de produto (ex: matriz de custo assimétrico do erro, estratégia de deploy serverless dimensionada ao estágio do negócio).
6.2. Profundidade do bloco 7 (MLOps): **3** — a seção 7 do plano nomeia o que monitorar (taxa de conversão real, razões de recusa, cruzamento entre diagnósticos emitidos e desfechos reportados) e dá uma frequência de monitoramento explícita ("processo manual mensal de revisão"), mas o gatilho de retreino permanece qualitativo — "reavaliação dos parâmetros do modelo quando houver queda perceptível na taxa de sucesso reportada" — sem um limiar numérico definido para o que conta como queda relevante, nem uma cadência própria de retreino (distinta da cadência de monitoramento).

   *Ponto de atenção (não penaliza esta nota, mas deve ser registrado):* o repositório `FGV-aulas-1` como um todo é majoritariamente um fork/clone do material de referência do curso (notebooks `01` a `13` de `aplicacoes-de-negocio/` e demais pastas são do professor, não produção do aluno). O único artefato autoral é o notebook `14-portabilidade-credito-lumina.ipynb` somado ao plano de projeto e à apresentação — o que é avaliado aqui. Recomenda-se, para fins de portfólio, isolar o projeto próprio num repositório dedicado, deixando claro o que é material do curso e o que é autoral.

**7. Correção técnica — 10/10**
7.1. Código executa sem erro: **5** — todas as 11 células de código têm `execution_count` sequencial (1→11), sem nenhuma saída de erro.
7.2. Split antes de pré-processamento: **5** — `StandardScaler().fit(X_train)` na seção 5 ("Separação treino/teste"), sem vazamento.
7.3. Métrica adequada à distribuição: **5** — AUC-ROC é apropriada para o desbalanceamento de ~26,7% de portabilidades vantajosas (impresso na seção 3, "Geração do dataset simulado").
7.4. Baseline no mesmo split: **5** — Regressão Logística e MLP usam `X_train_s`/`X_test_s` idênticos.

**8. Qualidade do código — 7,5/10**
8.1. Seeds fixadas: **5** — `np.random.seed(42)` global, `random_state=42` em split e modelos, e seeds explícitas (42, 7, 123) no bloco de validação multi-seed.
8.2. Dependências declaradas: **4** — célula `%pip install -q numpy pandas scikit-learn matplotlib tensorflow` explícita, mas instala `tensorflow` sem que o framework seja importado ou usado em nenhum ponto do notebook (a arquitetura final é `MLPClassifier` do scikit-learn) — dependência supérflua/enganosa.
8.3. Organização em funções/seções: **3** — bem seccionado com markdown numerado (0 a 10), mas a seção "Validação com múltiplas seeds" duplica quase integralmente (~40 linhas) a lógica de geração de dados já escrita na seção 3, em vez de extrair uma função reutilizável (`gerar_base_contratos(seed)`).

**9. Honestidade dos resultados — 7,5/10**
9.1. Múltiplas seeds/execuções: **5** — validação com 3 seeds (42, 7, 123), reportando média ± desvio padrão do AUC e reconhecendo explicitamente o "empate técnico" na seed 123 (seção "Resultado honesto: a vantagem do MLP e a estabilidade entre seeds").
9.2. Seção de limitações presente: **3** — não há uma seção dedicada de "Limitações" no notebook. Existe, de forma distribuída, o reconhecimento de que os dados são simulados (seção 3: "vamos simular 2.000 contratos") e que a Lumina "não possui histórico volumétrico para estabelecer baselines consolidados" (`plano-projeto-lumina.md`, seção 1), mas isso nunca é nomeado explicitamente como uma limitação do experimento nem discutido em termos de impacto na confiabilidade do AUC reportado.

## 4. Pontos fortes

- **9.1 Múltiplas seeds/execuções** (5/5): validação com 3 seeds (42, 7, 123) reportando média ± desvio padrão do AUC, reconhecendo explicitamente o "empate técnico" na seed 123 e decidindo pela Regressão Logística mesmo com o MLP vencendo no AUC médio — decisão de negócio madura, bem documentada na seção 10 do notebook.
- **7.2 Split antes de pré-processamento** (5/5): `StandardScaler().fit(X_train)` na seção 5, ajustado só no treino, mesmo split usado para baseline e modelo avançado, execução limpa do início ao fim.
- **6.1 Cobertura dos 7 blocos do template** (5/5): `plano-projeto-lumina.md` cobre os 7 blocos integralmente e com profundidade de produto real (matriz de confusão ligada a churn B2B, mensagens de explicabilidade em linguagem natural para o usuário final).
- **3.1 Discute alternativa de automação/regra simples** (5/5): o slide 1 do PPTX e a seção 2 do notebook justificam de forma explícita por que uma regra estática não captura a interação não linear (spread × saldo × prazo) proposital construída no rótulo — alternativa simples é discutida e descartada com argumento técnico, não por omissão.

## 5. Pontos de melhoria

- **2.4 Comparação custo vs. retorno** (1/5): nenhum payback, ROI% ou breakeven é calculado — é o item de maior peso na rubrica, e o projeto já tem em mãos os números de receita (R$ 15-40/funcionário/mês) e de economia por contrato (R$ 2.900-4.200) sem nunca fechar essa conta.
- **2.1/2.2 Custo de construção e sustentação** (1/5 e 2/5): sem estimativa de custo de dados/treino/infraestrutura inicial, e a rotina de sustentação descrita (revisão manual mensal, retreino orientado a feedback) nunca é traduzida em número de custo.
- **8.3 Organização em funções/seções** (3/5): duplicação de ~40 linhas entre a seção 3 (geração de dados, seed única) e o bloco de validação multi-seed, em vez de uma função reutilizável.
- **9.2 Seção de limitações presente** (3/5): não há seção dedicada de "Limitações" que discuta explicitamente o uso de dados 100% sintéticos (sem histórico real da Lumina) e o que isso implica para a confiança no AUC de 0,86 reportado.

## 6. Nota final

**6,7 / 10** — Projeto tecnicamente sólido, honesto e com bom domínio do trade-off performance-vs-explicabilidade (o ponto mais maduro do trabalho), mas a ausência quase total de fundamentação de ROI — mesmo já tendo os números de receita e economia disponíveis — é o que mais pesa contra a nota, já que viabilidade econômica é um dos dois critérios de negócio priorizados nesta rubrica.

**Nível de maturidade: PoC/protótipo.** O plano descreve uma arquitetura de deploy (serverless, Netlify Functions) e um processo de monitoramento mensal, mas sem qualquer estimativa de custo de construção ou sustentação (2.1/2.2), sem comparação com o retorno esperado (2.4), e o próprio gatilho de retreino (6.2) permanece qualitativo, sem limiar numérico — falta a base econômica para justificar a entrada em produção; o passo seguinte é fechar essas contas, não ainda um piloto controlado.

## 7. Task list para evoluir o trabalho

**1. Aderência ao negócio**
- [ ] **1.1 Métrica de sucesso nomeada como receita/custo (4/5):** o Bloco B é direto — "Qual métrica de negócio define sucesso? Não acurácia técnica — receita, custo evitado, tempo economizado, risco reduzido." Suas metas de "churn <10%" e "engajamento >60%" ainda são métricas de produto/retenção, não de receita. Você já tem os insumos para fechar essa conta: a faixa de receita B2B2C (R$ 15-40/funcionário/mês, seção 1 do notebook). O notebook `08-precificacao-dinamica.ipynb` (células 16-19) mostra a mesma lógica de tradução (`receita = preço × demanda_prevista`) — aplique de forma análoga: `receita_retida = ticket_B2B2C × nº de colaboradores × (1 - churn_evitado)`. — ver `plano-projeto-lumina.md` (seção 1)
- [ ] **1.3 Conexão entre métrica técnica e impacto de negócio (3/5):** a matriz de confusão (seção 6 do plano) já liga falso positivo/negativo a "risco de churn B2B" e a Conclusão do notebook (seção 10) traduz exemplos individuais em R$ (R$ 2.900 e R$ 4.200 por contrato). Falta agregar isso: o AUC de 0,86 nunca vira um número único de negócio para a carteira. O Bloco D chama isso de "Métrica-Alvo de Negócio: a métrica que traduz o resultado estatístico em dinheiro ou eficiência operacional" — como em `08-precificacao-dinamica.ipynb` (células 16-19), simule a economia projetada aplicando os R$ 2.900-4.200 por contrato ao volume esperado de contratos diagnosticados por mês. — ver seção 6 do plano e seção 10 do notebook (`14-portabilidade-credito-lumina.ipynb`)

**2. Viabilidade econômica (ROI)**
- [ ] **2.1 Custo de construção estimado (1/5):** o Bloco C trata isso diretamente — "Custo computacional — treino e inferência têm custo, meça contra o orçamento disponível" e "Tempo de treinamento — modelos complexos podem levar horas ou dias para treinar de novo." Mesmo a Regressão Logística final tendo custo de treino baixo, falta estimar o custo de coleta/validação de dados (cadastro manual + validação de faixas plausíveis, seção 2 do plano) e o custo de construir o pipeline de scoring serverless descrito na seção 7. — ver `plano-projeto-lumina.md` (seções 2 e 7)
- [ ] **2.2 Custo de sustentação estimado (2/5):** o Bloco B contrasta automação com IA — "atualiza-se a regra manualmente (automação) vs. retreina-se o modelo periodicamente (IA)" — que é exatamente o custo recorrente que falta quantificar: a seção 7 do plano já descreve o processo ("revisão manual mensal", "reavaliação dos parâmetros... quando houver queda perceptível"), mas nunca em termos de horas/custo de quem faz essa revisão mensal. — ver `plano-projeto-lumina.md` (seção 7, MLOps)
- [ ] **2.3 Retorno esperado com número (4/5):** já há números concretos (R$ 15-40/funcionário/mês e R$ 2.900/R$ 4.200 por contrato, seções 1, 6 e 10) — o Bloco D recomenda exatamente isso: "Traduza a complexidade técnica em valor financeiro e operacional." Falta só agregar num retorno total esperado do projeto (ex: nº de empresas clientes × colaboradores × ticket médio de economia). — ver seções 1 e 10 (`14-portabilidade-credito-lumina.ipynb`)
- [ ] **2.4 Comparação custo vs. retorno (1/5):** o Bloco C fecha esse raciocínio — "comece sempre por um baseline simples... um modelo mais complexo só se justifica se o ganho superar o custo." O curso não formaliza uma fórmula de payback/ROI%, mas depois de estimar 2.1/2.2 (custo) e 2.3 (retorno), uma fórmula genérica como `ROI = (retorno - custo) / custo` já é suficiente para fechar a conta que falta hoje. — ausente em todo o material do aluno

**6. Aderência ao template de projeto**
- [ ] **6.2 Profundidade do bloco 7/MLOps (3/5):** a seção 7 do plano já nomeia bem o que monitorar (taxa de conversão real, razões de recusa) e dá uma frequência de monitoramento ("processo manual mensal de revisão") — mas o gatilho de retreino continua qualitativo: "reavaliação dos parâmetros do modelo quando houver queda perceptível na taxa de sucesso reportada." Falta transformar isso num critério numérico acionável (ex: "retreinar se a taxa de sucesso cair mais de X p.p. em relação à baseline de lançamento") e definir uma cadência própria de retreino, distinta da cadência de monitoramento. — ver `plano-projeto-lumina.md` (seção 7, "Gatilho de retreino")

**8. Qualidade do código**
- [ ] **8.2 Dependências declaradas (4/5):** isso não é conteúdo do curso (é boa prática geral de ambiente/dependências, não de IA de negócio), mas a dica é direta: remova `tensorflow` da célula `%pip install`, já que a arquitetura final é `MLPClassifier` do scikit-learn e o framework nunca é importado nem usado — mantê-lo sugere uma stack de deep learning que não existe no notebook. — ver célula de instalação de dependências (`14-portabilidade-credito-lumina.ipynb`)
- [ ] **8.3 Organização em funções/seções (3/5):** isso também não é conteúdo do curso (é engenharia de software, fora do escopo de "Redes Neurais Aplicadas a Negócios"), mas vale como boa prática: extraia a geração da base de contratos simulados em uma função reutilizável (ex: `gerar_base_contratos(seed)`), eliminando a duplicação de ~40 linhas entre a seção 3 (geração de dados) e o bloco de validação multi-seed. — ver `14-portabilidade-credito-lumina.ipynb` (seção 3 e bloco "Validação com múltiplas seeds")

**9. Honestidade dos resultados**
- [ ] **9.2 Seção de limitações presente (3/5):** não há um trecho do material do curso específico sobre estrutura de seção de limitações a citar aqui — o reconhecimento de que os dados são simulados aparece disperso (seção 3 do notebook: "vamos simular 2.000 contratos"; `plano-projeto-lumina.md` seção 1: a Lumina "não possui histórico volumétrico para estabelecer baselines consolidados"), mas nunca é nomeado explicitamente como limitação do experimento nem discutido em termos de impacto na confiança do AUC de 0,86 reportado. Adicione uma seção curta e explícita reunindo esses dois pontos. — ver `14-portabilidade-credito-lumina.ipynb` (seção 10, "Conclusão")

## 8. Tópicos para o aluno revisar

- **ROI e viabilidade econômica de projetos de IA** (Bloco C) — motivado pela ausência de custo de construção/sustentação e de comparação custo-retorno em `plano-projeto-lumina.md`, apesar dos dados de receita (R$ 15-40/funcionário/mês) já estarem disponíveis no próprio texto.
- **Organização de código e reuso (DRY)** (Bloco C — Design de Projetos de IA) — motivado pela duplicação de ~40 linhas de geração de dados entre a seção 3 e o bloco de validação multi-seed em `14-portabilidade-credito-lumina.ipynb`.
- **Arquiteturas MLP e busca de hiperparâmetros** (Aula 2 — Arquitetura e Camada Oculta) — motivado pela escolha fixa de `hidden_layer_sizes=(16, 8)` na seção 7 do notebook, sem evidência de busca sistemática (grid/random search) entre arquiteturas alternativas, embora o plano de projeto (seção 5) mencione GridSearchCV como estratégia pretendida.
