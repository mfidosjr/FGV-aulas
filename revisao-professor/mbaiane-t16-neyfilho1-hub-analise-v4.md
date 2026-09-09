# Análise v4 — Ney Penalva Filho (`neyfilho1-hub`)

**Projeto:** O Fim da Evasão Inesperada de Talentos — People Analytics: Previsão de Turnover Voluntário com MLP
**Repositório:** [neyfilho1-hub/FGV-aulas](https://github.com/neyfilho1-hub/FGV-aulas)
**Arquivo analisado (v4, commit `33334759`):** `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`
**Nota v1:** 5,0/10 → **v2: 6,0/10** → **v3: 7,5/10** → **Nota v4: 6,0/10**

---

## 1. Abertura

Ney, os três pedidos pontuais da v3 foram implementados de fato: o notebook agora tem `execution_count` sequencial e outputs em todas as células, as legendas de ROI foram trocadas por f-strings dinâmicas, e o limiar de decisão passou a ter uma busca formal em Precision-Recall com justificativa. Do ponto de vista de estrutura de código, isso é um avanço real.

Mas ao verificar se os números **impressos** batem com o que o **código do próprio arquivo** produz, encontrei uma divergência que não consegui explicar por nenhuma causa técnica legítima — e que é mais grave do que a ausência de outputs da v3, porque desta vez há uma evidência de execução apresentada como prova, e ela não corresponde à execução real do código que a acompanha.

## 2. O que mudou desde a v3 (verificado com reexecução completa, do zero)

| Item da v3 | Situação na v3 | O que você fez na v4 | Verificado |
|---|---|---|---|
| **Sem outputs salvos** (3ª recorrência) | `execution_count: null` em todas as células | Outputs presentes, `execution_count` sequencial 1→11 | Presente na forma — mas ver ressalva central da seção 3. |
| Legendas hardcoded ("R$ 1,2M", "alinhado a 1,8-2 meses") | Texto fixo, desconectado do valor calculado | f-strings dinâmicas: `f'(~R$ {economia_bruta_ano/1e6:.2f}M)'`, payback sem faixa fixa | Estrutura do código correta — mas os valores que essas legendas dinâmicas exibem vêm de uma execução que não reproduz (seção 3). |
| Diagnóstico do PSI hardcoded ("Modelo Estável" sempre) | Texto fixo, ignorava o valor real | Função `diagnostico_psi()` com `if/elif` sobre os 3 limiares definidos na função de PSI | Lógica correta e condicional de fato — reexecutei e confirmei que responde certo a diferentes valores de PSI. |
| Threshold 0,35 sem justificativa | Constante arbitrária | Nova célula 9: varredura de 6 limiares (0,20 a 0,50) com Recall/Precisão/F1 por ponto, escolhendo o que maximiza F1 | Estrutura correta — mas a curva impressa não reproduz ao rodar o código (seção 3). |

## 3. O achado central: os outputs salvos não reproduzem a partir do código do próprio arquivo

Extraí o texto da célula 3 (geração de dados) e da célula 5 (regra heurística) **diretamente do JSON do arquivo `.ipynb`**, sem digitar nada manualmente, e executei em ambiente limpo (Python 3.9, numpy 1.26.4, pandas 2.3.3, scikit-learn 1.6.1). Esse trecho de código não tem nenhuma fonte de aleatoriedade não controlada — `np.random.seed(42)` é fixado no início da célula 3, e a regra heurística da célula 5 é uma função determinística das colunas geradas.

| Métrica | Valor "salvo" no notebook (output embutido no `.ipynb`) | Minha reexecução do código exato do arquivo (2 tentativas independentes) |
|---|---|---|
| Taxa de turnover | 21,0% (252 casos) | **21,0% (252 casos) — bate** |
| Recall da regra heurística | **35,3%** | **17,5%** |
| Recall da MLP (threshold 0,35) | **79,4%** | **66,7%** |
| Payback do investimento | **1,84 meses** | **2,19 meses** |

A taxa de turnover bate — o que já era esperado, pois é o único número que também bateu na v3. A partir da regra heurística (célula 5), porém, tudo diverge. Descartei explicações técnicas: comparei célula por célula o código de geração de dados entre a v3 e a v4 e a lógica de sorteio é idêntica (só comentários mudaram) — e a v3, mesmo sem outputs salvos, já reproduzia 17,5% quando eu a executei naquela rodada. Ou seja, o mesmo código determinístico, testado duas vezes em rodadas diferentes desta análise, sempre produz 17,5% — nunca os 35,3% que aparecem "salvos" no arquivo entregue como prova de execução.

Isso significa que o output embutido no `.ipynb` **não foi gerado por uma execução deste código**. Pode ter vindo de uma versão anterior e diferente do notebook (não commitada, como já ocorreu na v2 com os números do pptx da Aryadne) ou ter sido inserido sem execução real — mas, na prática, o efeito é o mesmo: o arquivo apresenta uma "prova de execução" que não é genuína, exatamente no ponto em que isso foi pedido de forma explícita e repetida (v1, v2 e v3).

Note que a divergência não invalida o mérito técnico da v3: quando executo o código de verdade, a MLP continua com desempenho saudável (AUC~0,92-0,93, ver v3) e o ROI continua internamente consistente (payback real ~2,2 meses, não os 13 meses da v2) — só os números específicos mostrados no arquivo entregue são outros, e não reproduzem.

## 4. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 7,0/10** *(v3: 9,0/10)*
1.3. Conexão entre métrica técnica e impacto de negócio: **3** *(v3: 5)* — a fórmula está certa e a legenda agora é dinâmica, mas o Recall que alimenta essa conta (79,4%) não reproduz a partir do código entregue.

**2. Viabilidade econômica (ROI) — 6,0/10** *(v3: 7,5/10)*
2.4. Comparação custo vs. retorno: **3** *(v3: 4)* — payback de 1,84 meses "salvo" no arquivo não reproduz (real: ~2,19 meses); a estrutura do cálculo está correta, só o número de entrada é não verificável.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 8,0/10** *(mantido, v3: 8,5/10)*

**4. ML tradicional vs. Redes Neurais — 9,0/10** *(v3: 10,0/10)* — comparação ainda presente e bem estruturada, mas os números específicos da tabela salva (Recall MLP 79,4%, RF entre 71-75%) não reproduzem; ao rodar de novo obtive MLP 66,7% e RF ~79-84% — ordem relativa entre modelos muda.

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 9,0/10** *(v3: 8,75/10)*
6.2. Profundidade do bloco de MLOps: **5** *(v3: 4)* — função `diagnostico_psi()` condicional é uma correção real e bem-feita, verificada respondendo certo a diferentes cenários.

**7. Correção técnica — 5,0/10** *(v3: 7,5/10)*
7.1. Código executa sem erro: **3** *(v3: 4)* — roda sem erro, mas os outputs salvos não correspondem à execução real desse mesmo código (seção 3) — o que é pior do que simplesmente não ter outputs.

**8. Qualidade do código — 7,5/10** *(v3: 6,7/10)* — dependências agora instaladas via `%pip install` executável (não mais comentado); estrutura da busca de threshold é limpa e bem documentada.

**9. Honestidade dos resultados — 2,0/10** *(v3: 5,0/10)*
Este é o achado que mais pesa na nota final. Foi pedido três vezes (v1, v2, v3) que o notebook fosse executado do início ao fim e os outputs salvos antes de comitar — precisamente para que a autenticidade dos números pudesse ser verificada sem depender só da minha reexecução. Na v4 isso finalmente aconteceu na forma, mas o conteúdo salvo não é uma execução genuína do código entregue — é uma regressão em relação à v3, que ao menos não afirmava nada sobre si mesma.

## 5. Nota final

**6,0 / 10** *(v3: 7,5/10)* — Há avanços reais e bem executados na estrutura do código desta rodada: a busca de threshold, as legendas dinâmicas e o diagnóstico condicional de PSI são implementações corretas, que reexecutei e confirmei que funcionam como pretendido quando alimentadas com dados reais. Mas a nota cai em relação à v3 porque o pedido mais básico e mais repetido de todo este ciclo — provar que o notebook foi executado de verdade — não só continua sem solução, como desta vez produziu uma evidência que não se sustenta: os outputs salvos no arquivo não reproduzem a partir do próprio código do arquivo, verificado duas vezes de forma independente. Isso pesa mais do que a ausência de outputs, porque transforma "faltou provar" em "a prova apresentada não é genuína".

**Nível de maturidade: PoC/protótipo.** O pipeline de modelagem, ROI e MLOps tem substância técnica real (confirmada pela minha própria reexecução), mas o projeto ainda não demonstrou, de forma verificável pelo revisor, que os números que ele próprio publica vêm da execução do código que ele próprio entrega.

## 6. O que preciso que você corrija para a v5

1. **Prioridade máxima:** rode o notebook com **kernel realmente limpo** (Restart & Run All no Jupyter/Colab, não apenas rodar células isoladas ou reaproveitar um kernel antigo) e confira, antes de comitar, se o número impresso na tela bate com o que está no arquivo salvo. O achado da seção 3 (Recall da regra heurística: 35,3% salvo vs. 17,5% real) é o tipo de coisa que uma conferência de 30 segundos pega.
2. Se você rodou uma versão diferente do notebook e depois editou o código sem re-executar, isso por si só já é a causa raiz — o hábito de "editar código depois de já ter os outputs salvos" é o que gera esse tipo de divergência.
3. Não é necessário refazer nada da lógica desta vez — a estrutura de threshold, ROI e PSI está correta. O único passo que falta é garantir que outputs = execução real do código ao lado, e essa é a mesma exigência das três rodadas anteriores.

Quando estiver pronto, me avise que faço uma nova revisão (v5) em cima da correção.
