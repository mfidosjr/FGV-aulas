# Handoff — Aulas FGV (redes neurais / MBA IA)

Documento de continuidade para outro agente assumir este trabalho. Escrito em 2026-08-20.

## Contexto geral

- Repositório: `mfidosjr/FGV-aulas`, local em `/Users/marcofidos/GitHub/FGV-aulas/`, remoto `origin` = `https://github.com/mfidosjr/FGV-aulas.git`, branch `main`.
- Autor/usuário: Marco Fidos, cientista de dados, preparando aulas de redes neurais para MBA da FGV.
- Convenções que devem ser seguidas (herdadas do CLAUDE.md global do usuário, repita-as sempre):
  - Responder sempre em **português brasileiro**.
  - Respostas curtas e diretas, sem resumo redundante ao final.
  - `git add` sempre **seletivo por nome de arquivo** — nunca `git add .` / `git add -A` (há risco real de pegar edições do próprio usuário que estão em andamento noutros arquivos — já aconteceu nesta sessão, ver "Cuidado" abaixo).
  - Commits em Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
  - **Nunca fazer push sem confirmação explícita**, salvo quando o usuário disser literalmente "commit e push".
  - Nunca fabricar/arredondar resultados de execução — todo número reportado nos notebooks tem que vir de execução real (ver seção "Ambiente de execução").

## Estrutura do repositório relevante

- `Aula 1 - RedesNeurais_202608_MBAIAN.ipynb` — em processo de renomeação PELO PRÓPRIO USUÁRIO no VS Code para `Aula 1-4 - RedesNeurais_202608_MBAIAN.ipynb` (working tree tinha isso como delete+untracked, não commitado). **Não commitar isso sem confirmação explícita do usuário** — é edição dele, não do agente.
- `Aula 2 - ArquiteturasRedesNeurais_202608_MBAIAN.ipynb` — aparecia como deletado no working tree (não commitado), provavelmente por causa da reestruturação acima. Também não tocar sem perguntar.
- `aplicacoes-de-negocio/` — pasta com 13 notebooks didáticos (ver lista abaixo), cada um comparando uma técnica de rede neural contra um baseline mais simples, com dados simulados.

### Os 13 notebooks de `aplicacoes-de-negocio/`

| # | Arquivo | Técnica NN | Baseline | Métrica principal |
|---|---|---|---|---|
| 01 | `01-deteccao-fraude.ipynb` | MLPClassifier | LogisticRegression (class_weight balanced) | precisão/recall/F1 (classe fraude) |
| 02 | `02-sistema-recomendacao.ipynb` | Embeddings/collaborative filtering (Keras) | média por item | RMSE |
| 03 | `03-previsao-demanda.ipynb` | SimpleRNN | média móvel | MAE |
| 04 | `04-chatbot-atendimento.ipynb` | TF-IDF + MLPClassifier | regra de palavras-chave | acurácia |
| 05 | `05-visao-computacional-varejo.ipynb` | CNN | limiar de intensidade | acurácia geral / ambíguos |
| 06 | `06-diagnostico-imagem-medica.ipynb` | CNN (disclaimer didático) | limiar | recall / acurácia |
| 07 | `07-manutencao-preditiva.ipynb` | LSTM | limiar de vibração | recall / precisão / antecedência |
| 08 | `08-precificacao-dinamica.ipynb` | MLPRegressor | LinearRegression | MAE / RMSE |
| 09 | `09-score-credito.ipynb` | MLPClassifier | LogisticRegression | AUC-ROC |
| 10 | `10-geracao-conteudo-marketing.ipynb` | GAN pequena | — (generativo) | estatísticas reais vs geradas |
| 11 | `11-prevencao-lavagem-dinheiro.ipynb` | Autoencoder | z-score | recall em FPR equalizado |
| 12 | `12-people-analytics-turnover.ipynb` | MLPClassifier | LogisticRegression | recall / AUC |
| 13 | `13-mini-llm-poucos-textos.ipynb` | Embedding+LSTM char-level (mini LLM) | — (não tem baseline) | geração de texto |

Todos os 13 já têm: célula 0 de `%pip install` (bootstrap de dependências), explicações de métrica após cada gráfico de comparação, e um anexo final "Outras Aplicações de Negócio com a Mesma Técnica" customizado por notebook.

## Ambiente de execução (IMPORTANTE)

Existem dois venvs Python com Jupyter kernel registrado:

1. **`fgv-aulas`** (kernel `"Python (FGV-aulas)"`) — venv **persistente** do usuário em `~/fgv-aulas-env/`, com numpy/pandas/scikit-learn/matplotlib/tensorflow/ipykernel. **Este é o kernel a usar** para qualquer reexecução — é o que o usuário seleciona no VS Code.
2. `nbviz-venv` — venv de scratchpad da sessão do agente anterior (`/private/tmp/claude-501/.../scratchpad/nbviz-venv`), **efêmero**, pode não existir mais em sessões futuras. Não depender dele.

Para reexecutar um notebook do zero de forma real (nunca fabricar outputs à mão):

```bash
/Users/marcofidos/fgv-aulas-env/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.kernel_name=fgv-aulas \
  --ExecutePreprocessor.timeout=300 \
  "/Users/marcofidos/GitHub/FGV-aulas/aplicacoes-de-negocio/<notebook>.ipynb"
```

Depois, validar sempre: JSON parseável, zero células com `output_type == "error"`.

Ao editar notebooks, usar a ferramenta `NotebookEdit` (não editar o JSON manualmente) — evita quebrar formatação e é mais seguro. Se editar via script Python/JSON diretamente, usar `indent=1` ao salvar (é o padrão que o resto do arquivo usa nesta pasta — checar sempre o indent original antes de reescrever, um mismatch de indent já causou diffs gigantes espúrios numa rodada anterior).

## Cuidado: edições do usuário em andamento

Durante esta sessão, o arquivo `Aula 1 - RedesNeurais_202608_MBAIAN.ipynb` teve edições feitas diretamente pelo usuário no VS Code, aparecendo como diffs no meio do trabalho do agente. Regra seguida: **nunca commitar esse arquivo junto com outro trabalho sem confirmação explícita** — ele pertence ao fluxo de edição do próprio usuário, não ao agente. Isso já causou um incidente nesta sessão: um `git commit` acabou incluindo uma renomeação de arquivo que já estava com `git add` feito por fora (provavelmente pela extensão de Git do VS Code) — foi corrigido com `git reset --soft HEAD~1` + `git restore --staged` seguido de recommit seletivo. **Sempre rodar `git status --short` antes de `git add`/`git commit` e conferir cada arquivo staged, mesmo quando você mesmo não rodou `git add -A`.**

## Linha do tempo do trabalho já feito (commits mais recentes primeiro)

- `39a6c44` — **test:** adiciona validação multi-seed (3 seeds: 42, 7, 123) nos 11 notebooks com comparação baseline-vs-NN (todos exceto 10 e 13). Ver resultados detalhados abaixo.
- `a38264c` — docs: reestruturação da Aula 1 pelo próprio usuário (seção de neurônio artificial).
- `397564c` — fix: notebook 01 teve o dataset aumentado de 3.000 para 20.000 transações (mesma taxa de fraude ~4%) para tentar sair do colapso total do MLP (0,000 em tudo). Resultado parcial: saiu do zero absoluto mas ainda muito atrás do baseline.
- `a7af4e7` — docs: explicações de métricas + anexos de outras aplicações de negócio, nos 13 notebooks.
- `643791b`, `69a1cc7`, `c7d0c6c` e anteriores — criação original dos 13 notebooks e da seção de aplicações de negócio na Aula 1.

## Plano: "5 melhorias" pedidas pelo usuário (TODOS CONCLUÍDOS)

O usuário pediu para implementar, **um a um**, 5 itens de evolução técnica/didática dos 13 notebooks. Os 5 foram concluídos nesta sessão (commits `39a6c44`, `d6acaaf`, `6ac5af1`/`e98ba87`, `2772ccd`, e o commit deste item 5 — ver histórico de commits no topo deste documento). Não há mais itens pendentes deste plano; qualquer trabalho futuro é novo, a combinar com o usuário.

### ✅ Item 1 — Robustez multi-seed (CONCLUÍDO, commit `39a6c44`)

Cada um dos 11 notebooks (01,02,03,04,05,06,07,08,09,11,12 — **excluindo 10 e 13**, que não têm comparação binária "vencedor") ganhou uma nova seção "Validação com múltiplas seeds" inserida entre a conclusão original e o anexo final, TOTALMENTE ADITIVA (nenhuma célula original foi alterada). Reexecuta o pipeline com seeds [42, 7, 123] e reporta média ± desvio-padrão.

**Resultados reais obtidos** (importante para não repetir trabalho nem inventar números):

| Notebook | A conclusão original do walkthrough (seed única) se sustenta com 3 seeds? | Números reais (média ± dp) |
|---|---|---|
| 01 fraude | Reforça — baseline estável; MLP muito instável | LogReg: prec 0,096±0,002 / rec 0,607±0,013 / F1 0,166±0,002. MLP: prec 0,182±0,315 / rec 0,016±0,028 / F1 0,029±0,051 (colapsa a 0,000 em 2 das 3 seeds) |
| 02 recomendação | **NÃO se sustenta** — quase empate, NN venceu em 2 de 3 seeds | Baseline RMSE 1,357±0,095; NN RMSE 1,350±0,130 |
| 03 previsão demanda | Mantém na média, mas RNN muito instável | Baseline MAE 33,04±0,79; RNN MAE 25,19±5,83 (seed 42 do multi-seed deu 31,63, diferente dos 20,45 do walkthrough original — ver nota abaixo) |
| 04 chatbot | Mantém direção, mas "100%" foi sorte da seed 42 | Baseline acc 0,867±0,058; NN acc 0,991±0,008 (cai a 98,7% fora da seed 42) |
| 05 visão varejo | **NÃO se sustenta** — inverte numa seed, CNN muito instável | Baseline: geral 0,969±0,008 / ambíguos 0,811±0,051. CNN: geral 0,941±0,041 / ambíguos 0,656±0,227 (varia de 40% a 83,3%!) |
| 06 diagnóstico médico | Mantém consistentemente | Baseline recall 0,542±0,037; CNN recall 0,882±0,043 |
| 07 manutenção preditiva | Mantém em recall/antecedência; empata em precisão | Baseline: recall 0,922±0,044 / precisão 0,923±0,024 / antecedência 5,00±0,42. LSTM: recall 0,878±0,048 / precisão 0,925±0,039 / antecedência 4,71±0,56 |
| 08 precificação | Mantém com folga, sem exceção | LinReg: MAE 54,067±1,283 / RMSE 73,382±2,508. MLP: MAE 9,868±2,005 / RMSE 15,080±3,800 |
| 09 score crédito | **NÃO se sustenta** — vantagem dentro do ruído, perde numa seed | LogReg AUC 0,875±0,031; MLP AUC 0,884±0,026 (MLP perde na seed 123: 0,898 vs 0,901) |
| 11 AML | Mantém, lição fica mais forte | z-score recall 0,62±0,08 (estável); autoencoder recall 0,44±0,27 (colapsa a 0,13 na seed 123) |
| 12 turnover | Mantém consistentemente, sem exceção | LogReg: recall 0,641±0,029 / AUC 0,924±0,015. MLP: recall 0,393±0,080 / AUC 0,870±0,031 |

**Nota técnica encontrada**: nos notebooks 02 e 03 (ambos com Keras/TensorFlow), rodar a seed 42 dentro do novo loop multi-seed produziu números **diferentes** dos já registrados no walkthrough original de seed única (ex.: notebook 03 deu MAE 31,63 no loop vs 20,45 no walkthrough original). Causa provável: a re-seed usada no loop (`tf.keras.utils.set_random_seed`) é mais abrangente que a usada originalmente (`tf.random.set_seed`), então o "mesmo" seed 42 não reproduz exatamente o mesmo estado aleatório. Isso NÃO foi corrigido — é uma limitação conhecida de determinismo em Keras/TF que pode valer a pena investigar/documentar no futuro, mas não bloqueia os outros itens.

### ⬜ Item 2 — Seção de limitações (PENDENTE)

Adicionar, em cada um dos 13 notebooks, uma seção explicitando que os dados são **simulados** (não reais) e quais simplificações isso implica, para não dar a impressão de que os resultados se transferem diretamente para produção. Deve ser aditivo (nova célula markdown), sem alterar texto existente. Sugestão de posição: perto do início (após a seção de contexto de negócio) ou perto da conclusão — decidir por notebook o que fica mais natural.

### ⬜ Item 3 — Célula de exercício aberto (PENDENTE)

Adicionar, em cada um dos 13 notebooks, uma célula de markdown ao final propondo um exercício/pergunta aberta para o aluno tentar por conta própria (ex.: "e se a taxa de fraude fosse 1% em vez de 4%? o que muda?"). Deve ser específico da técnica/domínio de cada notebook, não um exercício genérico copiado entre eles.

### ✅ Item 4 — Fix técnico do notebook 01 (CONCLUÍDO)

Causa raiz confirmada: `MLPClassifier` do scikit-learn não tem parâmetro `class_weight` (diferente da `LogisticRegression`, que usa `class_weight="balanced"`), então ele aprende a sempre prever a classe majoritária num dataset desbalanceado (~4% fraude). Fix implementado: nova seção "11. Corrigindo o colapso do MLP: balanceamento de classes no treino", que reamostra (com reposição, via `sklearn.utils.resample`) a classe minoritária **só no treino** até igualar a classe majoritária, mantendo o teste com a proporção real de fraude. Testado nas mesmas 3 seeds (42, 7, 123).

**Resultado real:** o oversampling elimina o colapso (não há mais 0,000 em nenhuma seed) e estabiliza o MLP — recall passa a ficar entre 0,498 e 0,530 nas 3 seeds (desvio-padrão cai de 0,028 para 0,016). O F1 do MLP corrigido (média 0,204) passa a **superar o do baseline (0,166)** nas 3 seeds, pois a precisão também melhora bastante (0,128 vs 0,096). Porém o **recall do baseline continua maior** (0,607 vs 0,512) — o baseline ainda captura mais fraudes reais, só que com muito mais falsos alarmes. Veredito: a correção resolve a instabilidade e torna o MLP competitivo (e melhor em F1), mas não o transforma em vencedor incondicional — a escolha entre os dois depende do custo relativo de falso positivo vs falso negativo que o negócio adota.

### ✅ Item 5 — Fix técnico do notebook 10 (CONCLUÍDO)

Fix implementado: nova seção "12. Melhorando o equilíbrio da GAN: LR menor no discriminador + label smoothing" — reconstrói gerador/discriminador do zero, reduz o LR do discriminador de 0,001 para 0,0002, e usa label smoothing (rótulos 0,9/0,1 em vez de 1,0/0,0) no treino do discriminador. Repete o mesmo treino de 60 épocas e compara com o original.

**Resultado real:** as losses passam a oscilar na mesma faixa (~0,58–0,71) em vez de divergir (0,03 vs 3,36) — sintoma visível resolvido. As estatísticas de imagem geradas ficam mais próximas das reais (diferença de média cai de 0,0512 para 0,0273, medido na mesma execução). Porém o discriminador corrigido fica enviesado na direção oposta: acerta 94% das imagens reais mas só 4% das falsas (acurácia geral ~49%, ou seja, passou a prever "real" quase sempre) — o desequilíbrio não desaparece, troca de direção. Veredito honesto registrado no notebook: a correção melhora a qualidade estatística do gerador e a dinâmica de treino, mas não é uma solução perfeita — ilustra por que GANs são difíceis de calibrar. Nota also registrada: o TensorFlow não é 100% determinístico entre sessões de kernel mesmo com seeds fixas (mesma limitação já vista nos notebooks 02/03/11 do item 1), então os números "originais" reexecutados nesta seção diferem levemente dos registrados nas seções 9/10/Exercício do notebook (0,0512 vs 0,0702).

## Como retomar

1. Ler este arquivo por completo.
2. Rodar `git status --short` e `git log --oneline -5` para confirmar que o estado do repo bate com o descrito aqui (pode ter mudado se o usuário continuou editando "Aula 1" nesse meio tempo).
3. Perguntar ao usuário se quer seguir para o Item 2, ou revisitar algo.
4. Ao implementar cada item, seguir o padrão já estabelecido: uma tarefa por notebook (pode paralelizar com subagentes, 1 por notebook), sempre ADITIVO (não alterar células/números já existentes), sempre reexecutar de ponta a ponta com o kernel `fgv-aulas` antes de considerar concluído, sempre `git status --short` antes de `git add` (seletivo, nunca `-A`), e sempre confirmar com o usuário antes de `git push`.
