# Tutorial — Como executar e conferir o notebook antes de comitar (para a v5)

Ney, isso complementa a análise v4. O achado central de lá foi que os números salvos no arquivo não batem com o que o código do arquivo realmente produz quando executado. A causa mais comum disso é **editar código depois de já ter rodado o notebook** — daí os outputs na tela ficam "presos" numa execução anterior, e quando você salva o arquivo, ele carrega esse número antigo junto do código novo.

O único jeito de garantir que isso não aconteça é sempre fazer um **"Restart & Run All"** (reiniciar o kernel do zero e rodar todas as células em ordem) logo antes de salvar/comitar — nunca rodar célula por célula fora de ordem, e nunca reaproveitar um kernel que já rodou uma versão anterior do código.

## Passo a passo

### 1. Identifique onde está rodando o notebook

```
┌─────────────────────────────┬─────────────────────────────┐
│         Jupyter (local)      │      Google Colab (nuvem)    │
│  Barra de menu no topo com:  │  Barra de menu no topo com:  │
│  File  Edit  View  Run       │  File  Edit  View  Insert    │
│  Kernel  ...                 │  Runtime  Tools  ...         │
└─────────────────────────────┴─────────────────────────────┘
```

### 2. Reinicie o kernel do zero e rode tudo em ordem

**No Jupyter (Notebook clássico ou JupyterLab):**

```
Menu superior:
  Kernel
   └─ Restart Kernel and Run All Cells...
        └─ [Confirmar]
```

**No Google Colab:**

```
Menu superior:
  Runtime (ou "Ambiente de execução")
   └─ Restart and run all
        └─ [Confirmar]
```

Isso é diferente de apertar "Run" célula por célula, ou de usar `Ctrl+Enter`/`Shift+Enter` várias vezes — se o kernel não for reiniciado antes, ele ainda carrega variáveis e estado de execuções anteriores, e o resultado pode não corresponder ao código atual do arquivo.

### 3. Espere terminar e confira visualmente

Enquanto roda, cada célula de código mostra um contador à esquerda:

```
In [*]:  ...   <- célula rodando agora (asterisco)
In [7]:  ...   <- célula já terminou, é a 7ª a ter rodado nesta execução
```

Se ao final QUALQUER célula ainda mostrar `In [ ]:` (colchete vazio) ou `In [*]:` parado, é sinal de que ela não rodou ou travou — não comite assim.

### 4. Confira o número mais importante do trabalho manualmente

Antes de salvar, abra a célula 5 (regra heurística) e confira o `print` de Recall que aparece na tela **contra** o que você está prestes a comitar. Foi exatamente esse número (35,3% salvo vs. 17,5% real) que não bateu na v4 — uma conferência de 10 segundos aqui já pega o problema.

```
Checklist antes de salvar/comitar:
  [ ] Fiz Restart Kernel / Restart Runtime (não só "Run")
  [ ] Rodei "Run All" logo em seguida, sem editar nada no meio
  [ ] Todas as células terminaram (nenhum "In [ ]" vazio, nenhum "In [*]" parado)
  [ ] O número de Recall da célula 5 na tela bate com o que vou comitar
  [ ] Salvei o arquivo (Ctrl+S / Cmd+S) DEPOIS de ver os outputs corretos na tela
```

### 5. Só então: commit e push

Se algum passo do checklist falhar, o problema mais provável é: o código foi editado depois da última execução completa. Nesse caso, repita o passo 2 antes de salvar.

---

Qualquer dúvida sobre esse processo específico, me avise — não precisa refazer nenhuma lógica do notebook para a v5, só garantir que a execução salva é a execução real.
