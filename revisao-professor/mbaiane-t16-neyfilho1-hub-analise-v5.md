# Análise v5 — Ney Penalva Filho (`neyfilho1-hub`)

**Projeto:** O Fim da Evasão Inesperada de Talentos — People Analytics: Previsão de Turnover Voluntário com MLP
**Repositório:** [neyfilho1-hub/FGV-aulas](https://github.com/neyfilho1-hub/FGV-aulas)
**Arquivo analisado (v5, commit `62efc4fa`):** `aplicacoes-de-negocio/12-people-analytics-turnover.ipynb`
**Nota v1:** 5,0/10 → v2: 6,0/10 → v3: 7,5/10 → v4: 6,0/10 → **Nota v5: 8,5/10**

---

## 1. Abertura

Ney, desta vez a verificação fechou. Extraí o código de todas as 11 células diretamente do JSON do arquivo (sem digitar nada manualmente) e reexecutei o notebook inteiro do zero, em ambiente limpo — **todos os números batem, célula por célula, com o que está salvo no arquivo**: a regra heurística (17,5%/42,7%/24,8%/77,8%), a busca de threshold, a tabela comparativa dos 4 modelos, a validação multi-seed nas 3 seeds, o ROI (payback de 2,19 meses) e o teste de PSI (0,0055 vs. 1,0824). Esta é a primeira vez, em 5 rodadas, que consigo confirmar isso de ponta a ponta sem nenhuma divergência.

Isso resolve o problema mais persistente de todo este ciclo — o mesmo que eu pedi na v1, repeti na v2 e na v3, e que na v4 tinha piorado (outputs presentes, mas fabricados). O tutorial de execução limpa que passei parece ter funcionado: os números aqui são, de fato, o que o seu código produz.

## 2. O que mudou desde a v4 (verificado com reexecução completa, do zero)

| Item da v4 | Situação na v4 | O que você fez na v5 | Verificado |
|---|---|---|---|
| Outputs salvos não reproduziam o código do arquivo (achado mais grave da v4) | Recall da regra: 35,3% salvo vs. 17,5% real; Recall MLP: 79,4% salvo vs. 66,7% real; payback: 1,84 salvo vs. 2,19 real | Kernel reiniciado e notebook rodado do início ao fim antes de salvar | Reexecutei as 11 células do zero, extraindo o código diretamente do `.ipynb`: **100% dos números batem** — regra heurística, threshold, comparativo de 4 modelos, multi-seed (3 seeds), ROI e PSI, todos idênticos ao que está salvo. |
| Duas cópias do notebook (`aplicacoes-de-negocio/` e `revisao-professor/`) | — | Ambas atualizadas juntas | Comparadas célula a célula: 0 diferenças entre as duas cópias. |

## 3. Achado novo, menor: a "otimização de threshold" tem uma leitura mais honesta que a frase sugere

A célula 9 varre 6 limiares (0,20 a 0,50) e imprime a frase "O limiar 0.35 maximiza o equilíbrio operacional". Mas a própria tabela impressa mostra que **Recall, Precisão e F1 são idênticos para os limiares 0,25, 0,30, 0,35, 0,40 e 0,50** (66,7%/73,7%/70,0% nos cinco) — só o limiar 0,20 difere (68,3%/72,9%/70,5%). Ou seja, não há um "ponto ótimo" isolado: existe um platô largo onde a probabilidade prevista pela MLP simplesmente não tem exemplos de teste caindo entre 0,25 e 0,50, então qualquer limiar nessa faixa dá exatamente o mesmo resultado. Isso não é um erro nem fabricação — é uma característica real da distribuição de probabilidades do modelo neste conjunto de teste (300 casos) — mas a frase de justificativa de negócio dá a entender que 0,35 foi escolhido por ser especificamente ótimo, quando na verdade ele só corresponde a um platô. Vale mencionar isso na conclusão, se quiser ser mais preciso.

## 4. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 9,0/10** *(mantido, v3: 9,0/10)*

**2. Viabilidade econômica (ROI) — 8,5/10** *(v4: 6,0/10)*
2.3/2.4. Retorno e payback: **5** *(v4: 3/4)* — R$ 1.020.000,00 e payback de 2,19 meses, agora confirmados como resultado de execução real e reproduzível do código entregue.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 8,5/10** *(mantido)*

**4. ML tradicional vs. Redes Neurais — 10,0/10** *(v4: 9,0/10)* — comparação entre 4 abordagens (regra, LR, RF, MLP), com números agora verificados como genuínos; a MLP continua atrás da Regressão Logística (AUC 0,928 vs. 0,956) e do Random Forest (Recall 66,7% vs. 85,7%/90,5%), e isso é discutido honestamente na seção de limitações.

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 9,0/10** *(mantido)*

**7. Correção técnica — 9,0/10** *(v4: 5,0/10)*
7.1. Código executa sem erro e outputs correspondem à execução real: **5** *(v4: 3)* — primeira vez, nas 5 rodadas, que a reexecução completa e independente bate 100% com o arquivo entregue.

**8. Qualidade do código — 8,0/10** *(v4: 7,5/10)*

**9. Honestidade dos resultados — 8,0/10** *(v4: 2,0/10)*
9.1. Múltiplas seeds/execuções: **5** — implementado e agora verificado como genuíno.
9.2. Seção de limitações: **4** — presente, reconhece o trade-off real entre LR e MLP.
Nota do critério sobe de forma acentuada porque o problema central das 4 rodadas anteriores — a autenticidade dos números publicados — finalmente foi resolvido e verificado de ponta a ponta. Não é 10 porque a frase de "limiar ótimo" (seção 3) ainda tem uma leitura um pouco mais favorável do que os dados sustentam, embora de forma muito mais branda que os achados anteriores.

## 5. Nota final

**8,5 / 10** *(v4: 6,0/10)* — O problema que atravessou as 4 rodadas anteriores deste projeto — não conseguir confirmar que os números publicados vêm de uma execução real do código entregue — foi finalmente resolvido, e verifiquei isso da forma mais rigorosa possível: reexecutando as 11 células do zero, extraindo o código diretamente do arquivo, e conferindo cada número contra o que está salvo. Todos batem. Combinado com os avanços técnicos já consolidados nas rodadas anteriores (MLP com desempenho saudável, ROI reconciliado, PSI com perturbação real e diagnóstico condicional, multi-seed genuíno), este é o primeiro momento do ciclo em que o trabalho pode ser avaliado com confiança total nos números que ele próprio apresenta.

**Nível de maturidade: PoC/protótipo validado.** A trilha de evidências agora é auditável de ponta a ponta — é exatamente o padrão que deveria ter sido entregue desde a v1.

## 6. Observação final (não bloqueante)

Não há pedido de v6. Se quiser refinar mais um ponto, o único item que resta é ajustar a frase da seção 5 do notebook para refletir que 0,35 está dentro de um platô de thresholds equivalentes (0,25 a 0,50), não um ponto isoladamente ótimo — mas isso não muda a nota nem é bloqueante.
