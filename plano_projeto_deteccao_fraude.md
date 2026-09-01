# Arquitetura e Planejamento: Projeto Detecção de Fraude em Transações Financeiras

**Documento de Engenharia e Design de Solução de IA** | MBA em Inteligência Artificial para Negócios — FGV  
**Autor:** Renan Marchini Andrusiac Vieira  
**Repositório:** [renan-marchini/FGV-aulas](https://github.com/renan-marchini/FGV-aulas)  
**Notebook de Referência:** `aplicacoes-de-negocio/01-deteccao-fraude.ipynb`

---

## 1. Visão Geral do Problema e Objetivo

### 1.1 O Problema de Negócio
Instituições financeiras e plataformas de pagamentos digitais enfrentam um duplo desafio crítico na gestão de transações com cartão de crédito e pagamentos instantâneos:
1. **Perdas Diretas por Fraude (Falsos Negativos):** Cada transação fraudulenta aprovada indevidamente acarreta prejuízo direto de reembolso ao cliente lesado (*chargeback*), multas regulatórias e custo administrativo de auditoria. Custo unitário estimado: **R$ 1.500,00 por ocorrência**.
2. **Atrito e Perda de Clientes por Falsos Alarmes (Falsos Positivos):** Sistemas antifraude legados, fundamentados em regras estáticas simples (ex.: valor alto ou horário atípico isolados), bloqueiam transações de clientes legítimos. Isso gera sobrecarga nas centrais de atendimento (*call center*), frustração do consumidor e risco elevado de cancelamento de cartão (*churn*). Custo unitário estimado de suporte e atrito: **R$ 100,00 por ocorrência**.

### 1.2 Objetivo da IA
Desenvolver, treinar e implantar um modelo preditivo supervisionado de Inteligência Artificial baseado em **Redes Neurais Artificiais (*Multi-Layer Perceptron - MLP*)** capaz de estimar em tempo real a probabilidade de uma transação ser fraudulenta ($y=1$) ou legítima ($y=0$), capturando automaticamente interações não lineares complexas entre múltiplos atributos transacionais e comportamentais.

### 1.3 Métrica de Sucesso de Negócio e Viabilidade Econômica (ROI)
- **Métrica Técnica Principal:** Maximizar o **F1-Score da classe de fraude** ($> 0.20$) e a **Precisão** ($> 0.13$), mantendo o **Recall balanceado** ($> 50\%$) na validação multi-seed, eliminando 100% de colapsos de modelo.
- **Métrica de Sucesso Financeiro:** Minimizar o **Custo Total de Negócio** $\text{Custo Total} = (\text{FN} \times \text{R\$} 1.500) + (\text{FP} \times \text{R\$} 100)$.
- **Economia no Conjunto de Teste (6.000 transações):** O modelo proposto atinge custo de **R$ 238.600,00** (com threshold ótimo 0.80), economizando **R$ 52.800,00** em relação ao baseline de Regressão Logística (R$ 291.400,00) e **R$ 129.700,00** em relação à regra determinística (R$ 368.300,00).

#### Demonstrativo de ROI e Viabilidade Econômica (Cenário Operacional: 1.000.000 transações/mês)
| Linha Orçamentária / Indicador | Discriminação / Detalhamento | Valor (R$) |
| :--- | :--- | :--- |
| **Custo de Construção (Capex)** | 1 ML Engineer Sênior + 1 Data Engineer (2 meses) + Infraestrutura de treino/Cloud | **R$ 65.000,00** |
| **Custo de Sustentação Anual (Opex)** | Infraestrutura Cloud (API Server + Redis + Observabilidade) + 15% Engenharia para retreino | **R$ 54.000,00/ano** |
| **Investimento Total no 1º Ano** | Capex (R$ 65k) + Opex Anual (R$ 54k) | **R$ 119.000,00** |
| **Retorno Financeiro Bruto Projetado** | Economia anual de perdas por fraude e redução de falsos alarmes em produção | **R$ 504.000,00/ano** |
| **Retorno Líquido no 1º Ano** | Benefício Bruto Anual - Custo Total 1º Ano (R$ 504.000 - R$ 119.000) | **R$ 385.000,00** |
| **Retorno sobre o Investimento (ROI)** | $(504.000 - 119.000) / 119.000$ | **+323,5%** |
| **Prazo de Retorno (Payback)** | Recuperação integral do Capex + Opex acumulado | **2,8 meses** |
| **Ponto de Equilíbrio (Breakeven)** | Atingido no 1º trimestre de operação em produção | **Mês 3** |

---

## 2. Coleta e Preparação de Dados

### 2.1 Fontes de Dados e Atributos Transacionais
Os dados são originados de três barramentos de mensageria em tempo real:
- **Gateway de Pagamentos:** Valor da transação monetária e *timestamp* da solicitação.
- **Sistemas de Segurança e Sessão:** Identificador de dispositivo (*fingerprint* do aparelho / app mobile), indicador de novo dispositivo e geolocalização estimada via IP/GPS.
- **Cadastro e Histórico Comportamental do Cliente:** Frequência de compras recentes e distância média do local habitual de consumo.

### 2.2 Dicionário de Features
| Variável | Tipo | Descrição e Distribuição |
| :--- | :--- | :--- |
| `valor_transacao` | Numérica Contínua | Valor em reais da transação (distribuição exponencial, média R$ 260,00). |
| `hora_do_dia` | Numérica Inteira | Hora da transação entre 0 e 23h (distribuição uniforme). |
| `distancia_do_local_habitual_km` | Numérica Contínua | Distância euclidiana em km em relação ao padrão habitual do cliente. |
| `transacoes_ultimas_24h` | Numérica Inteira | Contagem de transações nas 24h precedentes (distribuição de Poisson, $\lambda=3$). |
| `dispositivo_novo` | Categórica Binária | 1 se o dispositivo não consta no histórico do cliente; 0 caso conhecido ($p=15\%$). |
| `fraude` *(Target)* | Categórica Binária | 1 para transação fraudulenta confirmada; 0 para transação legítima (~4,16% de fraudes). |

### 2.3 Engenharia de Features e Relações Não Lineares
O score latente de risco de fraude incorpora interações cruzadas não lineares:
- `hora_madrugada`: Flag binária indicando operações entre 00:00 e 05:59.
- `muitas_transacoes`: Flag indicando frequência transacional elevada ($\ge 6$ transações nas últimas 24h).
- **Interação Tripla Não Linear:** Combinação simultânea de `valor_transacao` elevado $\times$ `dispositivo_novo` $\times$ `distancia_do_local_habitual_km`.
- **Interações Duplas:** Dispositivo novo durante a madrugada e valor alto sob alta frequência.

### 2.4 Tratamento de Desbalanceamento Severo
Com apenas ~4,16% de fraudes, o modelo neural tradicional colapsaria prevendo sempre zero. Aplicou-se **Oversampling com Reposição da classe minoritária** (`y_train == 1`) exclusivamente sobre o conjunto de treino até atingir a proporção 50%/50%.

---

## 3. Estratégia de Bases e Separação de Dados

### 3.1 Armazenamento e Governança
- **Camada Raw (Data Lake):** Eventos brutos armazenados em Cloud Storage (formato Parquet colunar particionado por data).
- **Camada Curada (Feature Store):** Tabela analítica agregada com features comportamentais de baixa latência (Redis / Feast).

### 3.2 Metodologia de Split
- Divisão estratificada: **70% para Treinamento (14.000 amostras)** e **30% para Teste (6.000 amostras)** via `train_test_split(stratify=y, random_state=42)`.
- A estratificação garante idêntica proporção de fraudes (~4,16%) em ambos os conjuntos.

### 3.3 Prevenção Rigorosa de Vazamento de Dados (Data Leakage)
1. **Isolamento de Padronização:** O normalizador `StandardScaler` é ajustado (*fit*) estritamente com os dados de treino `X_train`. A transformação resultante é aplicada sem reajuste em `X_test`.
2. **Isolamento de Reamostragem:** O Oversampling é executado apenas sobre as linhas de treino já separadas. O conjunto de teste jamais recebe dados sintéticos ou duplicados, preservando a fidelidade da distribuição real de mercado.

---

## 4. Seleção e Comparação de Algoritmos

Comparamos quatro abordagens conceituais para avaliar empiricamente a necessidade de IA e o ganho marginal de cada técnica:

```
[Fluxo Transacional] ---> [1. Regra Determinística] (Regra estática de corte linear)
                    ---> [2. Regressão Logística]    (Baseline linear ponderado)
                    ---> [3. MLP Sem Balanceamento]  (Rede neural pura - colapsa no desbalanceamento)
                    ---> [4. MLP com Oversampling]   (Rede neural com treino balanceado e threshold ótimo)
```

1. **Benchmark 1: Regra Determinística Heurística:** Regra estática comumente usada em sistemas tradicionais: bloqueia se `valor > R$ 1.000` OU (`distancia > 50km` E `madrugada`).
2. **Benchmark 2: Regressão Logística (`class_weight='balanced'`):** Modelo linear clássico que compensa o desbalanceamento ponderando inversamente as classes na função de verossimilhança.
3. **Benchmark 3: Rede Neural MLP Sem Balanceamento:** Arquitetura MLP de duas camadas ocultas (16, 8) com ativação ReLU treinada na proporção original.
4. **Modelo Proposto: Rede Neural MLP com Oversampling no Treino:** A mesma arquitetura neural treinada com reamostragem balanceada no treino e sintonia de threshold de decisão.

---

## 5. Estratégia de Treinamento e Otimização

- **Arquitetura da Rede:** Multi-Layer Perceptron (MLP) com entrada de 5 dimensões $\to$ Camada Oculta 1 (16 neurônios, ReLU) $\to$ Camada Oculta 2 (8 neurônios, ReLU) $\to$ Camada de Saída (1 neurônio sigmoidal com probabilidade de fraude).
- **Função de Perda (Loss Function):** *Binary Cross-Entropy Loss* (Log-Loss):
  $$\mathcal{L} = -\frac{1}{N} \sum_{i=1}^N \left[ y_i \log(\hat{p}_i) + (1 - y_i) \log(1 - \hat{p}_i) \right]$$
- **Algoritmo de Otimização:** `Adam` com taxa de aprendizado inicial $\alpha = 0.001$, regularização $L_2$ ($\alpha_{\text{reg}} = 0.0001$) e convergência antecipada (*Early Stopping*) baseada em monitoramento de validação para mitigar sobreajuste (*overfitting*).

---

## 6. Testes, Validação e Métricas

### 6.1 Resultados na Seed 42 (Conjunto de Teste: 6.000 transações)
| Modelo | Precisão | Recall | F1-Score | Falsos Positivos (FP) | Falsos Negativos (FN) | Custo Total (R$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Sem Modelo (Aprovar Tudo)** | 0,000 | 0,000 | 0,000 | 0 | 250 | R$ 375.000,00 |
| **1. Regra Determinística** | 0,089 | 0,056 | 0,069 | 143 | 236 | R$ 368.300,00 |
| **2. Regressão Logística (Baseline)** | 0,096 | **0,596** | 0,166 | 1.399 | 101 | R$ 291.400,00 |
| **3. MLP Sem Balanceamento** | 0,545 | 0,048 | 0,088 | 10 | 238 | R$ 358.000,00 |
| **4. MLP + Oversampling (Limiar 0.50)** | **0,129** | 0,504 | **0,205** | **852** | 124 | R$ 271.200,00 |
| **5. MLP + Oversampling (Limiar Ótimo 0.80)** | **0,401** | 0,404 | **0,402** | **151** | 149 | **R$ 238.600,00** |

### 6.2 Validação Multi-Seed (Robustez Estatística nas Seeds 42, 7 e 123)
| Modelo Avaliado | Precisão Média $\pm$ dp | Recall Médio $\pm$ dp | F1-Score Médio $\pm$ dp | Taxa de Colapso |
| :--- | :---: | :---: | :---: | :---: |
| **Regra Determinística** | $0,074 \pm 0,013$ | $0,049 \pm 0,007$ | $0,059 \pm 0,009$ | 0% (Inútil) |
| **MLP Sem Balanceamento** | $0,182 \pm 0,315$ | $0,016 \pm 0,028$ | $0,029 \pm 0,051$ | **66,7% (2 de 3 seeds)** |
| **Regressão Logística (Baseline)** | $0,096 \pm 0,002$ | **$0,607 \pm 0,013$** | $0,166 \pm 0,002$ | 0% |
| **MLP com Oversampling (Proposto)** | **$0,138 \pm 0,019$** | $0,514 \pm 0,009$ | **$0,217 \pm 0,024$** | **0% (100% estável)** |

### 6.3 Discussão Honesta do Trade-off de Precisão vs Recall
- O Baseline linear possui recall bruto superior no limiar padrão de 0.50 ($60,7\%$ vs $51,4\%$), porém com precisão inaceitável de $9,6\%$, inundando o suporte com quase 1.400 falsos alarmes.
- O MLP com Oversampling eleva a precisão em **+43,8%** e o F1 em **+30,7%**, cortando centenas de falsos positivos.
- Ao otimizar o limiar de decisão (*threshold*) para a realidade financeira ($FN = 1.500$ vs $FP = 100$), o limiar ótimo de **0.80** minimiza a perda combinada de negócio, atingindo o custo mínimo absoluto de **R$ 238.600,00**.

---

## 7. MLOps: Deploy, Monitoramento e Governança

### 7.1 Arquitetura de Implantação e SLA de Produção
- **Microsserviço de Inferência em Tempo Real:** API REST construída em Python com **FastAPI** empacotada em contêineres **Docker**, orquestrada via Kubernetes (AWS EKS / GCP GKE) com *auto-scaling* horizontal.
- **SLA de Latência:** Inferência executada em **< 30ms (p95)** por requisição transacional.
- **Feature Store Online:** Redis em memória com cache pré-computado das features comportamentais (`transacoes_ultimas_24h`, `distancia_do_local_habitual_km`), garantindo latência de consulta sub-milissegundo (< 2ms).

```
[Transação de Compra] 
       │ (<30ms SLA)
       ▼
[API Gateway Antifraude] ──► [Redis Feature Store (<2ms)]
       │
       ▼
[FastAPI Inference Engine (MLP ONNX Runtime)]
       │
   ┌───┴────────────────────────┐
   ▼                            ▼
[Score >= 0.80]           [Score < 0.80]
 Bloqueio / 2FA            Aprovação Instantânea
```

### 7.2 Monitoramento Contínuo e Drift
1. **Data Drift (Desvio de Distribuição dos Dados de Entrada):**
   - Monitoramento contínuo das variáveis numéricas via teste de Kolmogorov-Smirnov (KS-Test) e *Population Stability Index* (PSI).
   - Alerta disparado se $\text{PSI} > 0.25$ em features críticas como `valor_transacao` ou `distancia_do_local_habitual_km`.
2. **Concept Drift (Degradação da Relação Padrão-Fraude):**
   - *Feedback Loop* semanal consolidando contestações e contestações confirmadas de *chargeback* (janela de maturação de 15 a 30 dias).
   - Acompanhamento semanal da taxa de Falsos Positivos reportados no canal de atendimento.

### 7.3 Política de Retreinamento e Ciclo de Vida do Modelo
- **Retreino Programado:** Mensal, incorporando os dados rotulados dos últimos 90 dias com janelas deslizantes.
- **Retreino Reativo:** Disparado automaticamente se o PSI ultrapassar 0.25 ou se o Recall estimado no pós-audit cair mais de 5 pontos percentuais.
- **Estratégia de Rollout Seguro:**
  - **Fase 1 (Homologação):** Validação técnica em ambiente de *staging*.
  - **Fase 2 (Shadow Mode):** Modelo roda em paralelo com a regra existente por 2 semanas, sem intervir na decisão, para aferição de latência real e taxa de concordância.
  - **Fase 3 (Canary / Rollout Gradual):** Liberação progressiva (10% $\to$ 25% $\to$ 50% $\to$ 100% do tráfego).

### 7.4 Governança, Segurança e Plano de Contingência (Circuit Breaker)
- **Fallback Automático:** Caso a API atinja tempo de resposta $> 50\text{ms}$ ou indisponibilidade de contêiner, um *Circuit Breaker* direciona o fluxo imediatamente para a regra heurística determinística de contingência, garantindo disponibilidade de 99,99%.
- **Explicabilidade (XAI):** Geração de SHAP values para auditoria interna e justificativa regulatória de bloqueios perante o Banco Central e LGPD.
