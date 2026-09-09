# Análise v3 — Ney Penalva Filho (`neyfilho1-hub`)

**Projeto:** O Fim da Evasão Inesperada de Talentos — People Analytics: Previsão de Turnover Voluntário com MLP
**Repositório:** [neyfilho1-hub/FGV-aulas](https://github.com/neyfilho1-hub/FGV-aulas)
**Arquivo analisado (v3, commit `c6f2ddad`):** `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`
**Nota v1:** 5,0/10 → **Nota v2: 6,0/10** → **Nota v3: 7,5/10**

---

## 1. Abertura

Ney, os dois pontos que eu marquei como "prioridade máxima" na v2 foram endereçados de verdade, e isso é o que mais importa aqui. A rede neural deixou de ter desempenho pior que aleatório, e a taxa de turnover — a premissa de todo o caso de negócio — agora bate com o que o código realmente gera. Reexecutei o notebook inteiro do zero para confirmar isso, e os números batem.

Só que, ao reexecutar, encontrei um padrão novo que precisa da sua atenção: em três pontos do notebook, os números que o código calcula agora são reais e reproduzíveis, mas os textos de interpretação ao lado deles ("R$ 1,2M", "alinhado a 1,8-2 meses", "Modelo Estável") ficaram travados de uma versão anterior e não foram atualizados para refletir o que o código passou a produzir. E o notebook chegou, pela terceira vez, sem nenhuma evidência de execução própria salva.

## 2. O que mudou desde a v2 (verificado com reexecução completa, do zero)

| Item da v2 | Situação na v2 | O que você fez na v3 | Verificado |
|---|---|---|---|
| **3.1** MLP com AUC=0,534 (pior que aleatório) | `early_stopping` padrão monitorando acurácia sob classe minoritária de 5,8%, sem `class_weight` | Reduziu `early_stopping=False`, ajustou arquitetura (32→16), `alpha=0,005`, `learning_rate_init=0,003`, `max_iter=400` | Reexecutei: AUC da MLP sobe para 0,928/0,918/0,925 nas seeds 42/123/999 — correção real e substancial. |
| **3.2** Taxa de turnover alegada (21%) não batia com a real (5,8%) | Intercepto calibrado pela média de `prob_turnover`, não pela fração acima do limiar 0,50 | Novo corte por percentil: `limiar_corte = np.percentile(prob_turnover, 79.0)` | Reexecutei: taxa real = 252/1200 = **21,0% exato** — bate com o alegado. |
| **3.2** Recall da MLP alegado (83%) não batia com o real (52,9%) | Número hardcoded na célula de ROI, sem vínculo com a execução | ROI agora usa `recall_real_mlp = recall_score(y_test, y_pred_mlp)` diretamente | Reexecutei: recall real = 66,7%, e é esse mesmo número que aparece na tabela de ROI — consistência interna restaurada. |
| **3.3** PSI comparava os dados de teste contra si mesmos (PSI=0 garantido) | Sem perturbação real | Dois cenários: baseline com ruído gaussiano leve, e "crise" com `Horas_Extras=1` para todos + queda de `Satisfacao_Clima` | Reexecutei: PSI baseline=0,19, PSI crise=1,08 — a função agora diferencia cenários de verdade. Mas ver ressalva na seção 3.2. |
| **9.1** Sem múltiplas seeds | Ausente | 3 seeds (42, 123, 999), split/scaler/treino refeitos por seed | Executado sem vazamento; valores variam de forma plausível entre seeds, sem padrão degenerado. |
| **9.2** Sem seção de limitações | Ausente | Seção 11 adicionada (natureza sintética, trade-off LR vs. MLP, dependência de ação humana, LGPD/human-in-the-loop) | Presente e coerente com os achados reais do notebook. |
| **8** Sem célula de instalação de dependências | Ausente | Célula 0 com `# !pip install` comentado | Presente, mas ainda comentada — não executa de fato num ambiente novo sem intervenção manual. |

## 3. Os dois pontos que ainda precisam de atenção

### 3.1 Terceira vez sem nenhuma evidência de execução própria

Todas as 11 células de código chegaram com `execution_count: null` e zero outputs salvos — exatamente como na v1 e na v2. Não há como saber, a partir do arquivo entregue, se você alguma vez rodou esta versão do notebook até o fim antes de commitar. Tudo que reporto aqui vem da minha própria reexecução, não da sua. Este é o único item pedido três vezes seguidas (v1, v2, v3) que nunca foi atendido.

### 3.2 Números reais, mas rótulos de interpretação hardcoded que não acompanham o que o código calcula

Este é um achado novo, e é mais sutil que uma fabricação — mas é do mesmo gênero: em três lugares do notebook, o texto que "traduz" um número para o leitor foi escrito como string fixa, não como algo condicionado ao valor calculado. Nos três casos, isso significa que se o código produzir um resultado pior, o texto ao lado continua dizendo que está tudo bem:

| Local | Número real (minha reexecução) | Rótulo hardcoded no print/tabela | Problema |
|---|---|---|---|
| Célula 17 (ROI) — economia bruta anual | R$ 1.020.000,00 | `"(R$ 1,2M)"` | Diverge ~18% do valor real ao lado |
| Célula 17 (ROI) — payback | 2,19 meses | `"(alinhado a 1,8-2 meses)"` | O valor real fica fora da faixa citada como referência |
| Célula 19 (PSI) — cenário normal | PSI = 0,1922 | `"Modelo Estável (PSI < 0.10)"` | Pela própria função que você escreveu (0,10 ≤ PSI < 0,25 = "Alerta de Monitoramento"), 0,19 não é "estável" — o rótulo contradiz o critério definido duas células acima |

O terceiro caso é o mais sério: o cenário que deveria representar "operação normal, sem crise" já dispara, pelos seus próprios critérios, um alerta de monitoramento — e isso fica escondido atrás de um texto que diz o contrário. Isso não muda a conclusão qualitativa do teste de drift (o cenário de crise ainda é claramente pior, PSI=1,08 vs. 0,19), mas mostra que o hábito de escrever a interpretação como texto fixo — em vez de derivá-la do número, com um `if`/`f-string` condicional — é o mesmo tipo de desconexão entre "o que o código calcula" e "o que o leitor vê" que gerou os problemas mais graves da v1 e v2. A diferença é que desta vez o número em si está certo; só a legenda está desatualizada.

## 4. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 9,0/10** *(v2: 7,5/10)*
1.3. Conexão entre métrica técnica e impacto de negócio: **5** *(v2: 2)* — a derivação (célula 17) agora usa o Recall real da MLP, calculado na própria execução, não mais um número hardcoded.

**2. Viabilidade econômica (ROI) — 7,5/10** *(v2: 5,0/10)*
2.3. Retorno esperado com número: **4** *(v2: 2)* — R$ 1.020.000,00 real e rastreável ao código, mas a legenda "(R$ 1,2M)" ao lado não foi atualizada (seção 3.2).
2.4. Comparação custo vs. retorno: **4** *(v2: 1)* — payback de 2,19 meses real e coerente com o Capex/Opex declarados; a legenda "alinhado a 1,8-2 meses" é o único ponto que destoa (seção 3.2).

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 8,5/10** *(v2: 7,5/10)*
3.1. Regra heurística comparada de forma justa, agora num cenário onde a MLP não está mais artificialmente derrotada por um bug de treino.

**4. ML tradicional vs. Redes Neurais — 10,0/10** *(mantido, v2: 10,0/10)* — Regressão Logística, Random Forest e MLP comparados nas mesmas condições, com a MLP agora competitiva (AUC~0,92-0,93), ainda que consistentemente abaixo da Regressão Logística (AUC~0,95-0,96) nas 3 seeds — um resultado honesto, discutido na seção de limitações.

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 8,75/10** *(v2: 6,25/10)*
6.2. Profundidade do bloco de MLOps: **4** *(v2: 3)* — teste de PSI agora usa perturbação real (seção 2), mas a mensagem de diagnóstico do cenário-base está errada (seção 3.2).

**7. Correção técnica — 7,5/10** *(v2: 6,9/10)*
7.1. Código executa sem erro: **4** *(mantido)* — roda do início ao fim quando executado por mim, mas chegou pela 3ª vez sem nenhuma evidência própria de execução (seção 3.1).
7.3. Métrica adequada à distribuição: **4** *(v2: 1)* — o problema de calibração da v2 foi corrigido; a taxa de turnover e o recall da MLP agora batem com a execução real.

**8. Qualidade do código — 6,7/10** *(v2: 5,8/10)* — célula de instalação de dependências adicionada (ainda que comentada); estrutura geral mais limpa que a v2.

**9. Honestidade dos resultados — 5,0/10** *(v2: 0,0/10)*
9.1. Múltiplas seeds/execuções: **4** *(v2: 1)* — implementado, 3 seeds, sem vazamento.
9.2. Seção de limitações: **4** *(v2: 1)* — presente e reconhece o trade-off real entre LR e MLP.
Nota geral do critério puxada para baixo pelo padrão da seção 3.2 (rótulos que não acompanham os números) e pela ausência de outputs salvos pela 3ª vez seguida (seção 3.1) — os dois problemas centrais de honestidade da v2 (MLP fabricada como vencedora, ROI sobre número fictício) foram genuinamente resolvidos, mas o hábito de escrever texto interpretativo desconectado do cálculo real ainda não foi corrigido, só mudou de forma.

## 5. Nota final

**7,5 / 10** *(v2: 6,0/10)* — Os dois problemas que eu classifiquei como prioridade máxima na v2 — a MLP com desempenho pior que aleatório e a taxa de turnover que não batia com a execução — foram corrigidos de verdade, e confirmei isso reexecutando o notebook inteiro do zero. Isso é o salto mais importante do trabalho até aqui. A nota não sobe mais porque um padrão relacionado, ainda que mais brando, se repete: números agora corretos acompanhados de textos de interpretação que não foram atualizados junto (seção 3.2), e a ausência de outputs salvos pela terceira rodada consecutiva (seção 3.1) — que é precisamente o que teria permitido pegar essas duas legendas desatualizadas antes de mim.

**Nível de maturidade: PoC/protótipo avançado.** A comparação entre modelos, o ROI e o teste de drift agora têm substância real por trás; falta o hábito de revisar o próprio output antes de entregar — o que teria pego sozinho os dois rótulos desatualizados da seção 3.2.

## 6. O que preciso que você corrija para a v4

1. **Rode o notebook do início ao fim com kernel limpo e salve os outputs antes de comitar.** Este pedido já foi feito na v1 e na v2 — é o único item ainda pendente das duas rodadas anteriores, e teria evitado o achado da seção 3.2 se você tivesse lido o próprio output impresso.
2. **Corrija as duas legendas hardcoded da célula de ROI** — troque `"(R$ 1,2M)"` e `"(alinhado a 1,8-2 meses)"` por texto derivado dos próprios valores calculados (ex: `f'({economia_bruta_ano/1e6:.2f}M)'`), para que nunca fiquem descoladas do número real.
3. **Corrija a mensagem de diagnóstico do PSI baseline** (célula 19) para refletir o critério que você mesmo definiu na função — hoje ela sempre imprime "Modelo Estável", independente do valor calculado. Vale usar um `if`/`elif` sobre o próprio `psi_normal`.
4. **Opcional, não bloqueante:** documente como o limiar de decisão `limiar_rh = 0,35` foi escolhido — hoje é uma constante fixa comentada como "para maximizar Recall", mas sem uma busca (ex: curva de Precisão-Recall) que sustente esse valor específico.

Quando estiver pronto, me avise que faço uma nova revisão (v4) em cima da correção.
