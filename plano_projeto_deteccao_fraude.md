# Arquitetura e Planejamento: Projeto Detecção de Fraude em Transações Financeiras

## 1. Visão Geral do Problema e Objetivo
- **O Problema:** Bancos e instituições financeiras enfrentam perdas milionárias anuais decorrentes de fraudes em cartão de crédito e transações digitais. Além do prejuízo direto de reembolso ao cliente, sistemas de detecção rígidos baseados em regras simples geram altas taxas de Falsos Positivos (bloqueio indevido de clientes legítimos), gerando atrito no suporte e insatisfação.
- **Objetivo da IA:** Treinar um modelo de inteligência artificial supervisionado capaz de prever se uma transação financeira é **fraudulenta (classe 1)** ou **legítima (classe 0)** a partir de atributos comportamentais e contextuais em tempo real.
- **Métrica de Sucesso (Negócio):** Maximizar a captura de fraudes (*Recall* > 50%) mantendo a precisão competitiva (*F1-Score* > 0.20), reduzindo o custo financeiro combinado de perdas por fraude e atendimento a falsos alarmes.

---

## 2. Coleta e Preparação de Dados
- **Fontes de Dados:** Sistemas transacionais em tempo real (Gateway de Pagamentos, Logs de Sessão de Dispositivo, Cadastro do Cliente).
- **Atributos Principais:**
  - `valor_transacao`: Valor monetário em reais (distribuição exponencial).
  - `hora_do_dia`: Hora da transação (0 a 23h).
  - `distancia_do_local_habitual_km`: Distância em km do local habitual de compra.
  - `transacoes_ultimas_24h`: Frequência de transações recentes (distribuição Poisson).
  - `dispositivo_novo`: Indicador binário (1 = novo dispositivo, 0 = conhecido).
- **Engenharia de Features:**
  - Normalização MinMax para intervalo [0, 1].
  - Variável derivada `hora_madrugada` (entre 0h e 5h).
  - Variável derivada `muitas_transacoes` (>= 6 transações em 24h).
  - Termos de interação não-linear complexos (ex.: `valor_norm * dispositivo_novo * dist_norm`).
- **Tratamento de Desbalanceamento:** O conjunto de dados apresenta apenas **~4% de fraudes**. Aplicou-se **Oversampling (reamostragem com reposição)** no conjunto de treinamento para impedir o colapso do modelo em prever apenas a classe majoritária.

---

## 3. Estratégia de Bases e Separação de Dados
- **Armazenamento:** Data Lake / Data Warehouse (ex.: BigQuery / Cloud Storage).
- **Metodologia de Split:** Divisão de **70% para Treino** e **30% para Teste** mantendo a proporção de classes via *Stratified Train-Test Split*.
- **Prevenção de Vazamento (Data Leakage):** O `StandardScaler` e a técnica de Oversampling são ajustados **exclusivamente no conjunto de treino** (`X_train`), sendo aplicados de forma estritamente transformativa no conjunto de teste (`X_test`).

---

## 4. Seleção de Algoritmos
- **Modelo Baseline:** Regressão Logística com `class_weight='balanced'`. Fornece um benchmark linear interpretável.
- **Modelo Avançado (Proposto):** Rede Neural *Multi-Layer Perceptron* (`MLPClassifier`) com arquitetura de 2 camadas ocultas `(16, 8)` e ativações não-lineares `ReLU`.
- **Justificativa:** A Rede Neural consegue aprender automaticamente combinações e interações não-lineares de alta ordem (ex.: valor alto + dispositivo novo + distância longa + madrugada) que o modelo linear isolado não consegue capturar.

---

## 5. Estratégia de Treinamento e Otimização
- **Função de Custo (Loss Function):** *Log-Loss* (Cross-Entropy Loss) otimizada via algoritmo `Adam`.
- **Hiperparâmetros:** `hidden_layer_sizes=(16, 8)`, `activation='relu'`, `max_iter=500`, `early_stopping=True`.
- **Balanceamento de Treino:** Duplicação randômica da classe minoritária no treino para equilibrar o gradiente de retropropagação.

---

## 6. Testes, Validação e Métricas
- **Validação Multi-Seed (Seeds 42, 7, 123):**
  - **Baseline (Regressão Logística):** Precisão = 0.096 ± 0.002 | Recall = 0.607 ± 0.013 | F1-Score = 0.166 ± 0.002
  - **Rede Neural MLP (Sem Balanceamento):** Colapsa em 0.000 em 2 de 3 seeds (Instável).
  - **Rede Neural MLP (Com Oversampling):** Precisão = **0.138 ± 0.019** | Recall = **0.514 ± 0.009** | F1-Score = **0.217 ± 0.024**
- **Conclusão:** O MLP com Oversampling elimina a instabilidade, entrega o maior F1-Score e aumenta a precisão em 43% em relação ao baseline.
- **Otimização de Limiar de Decisão (Matriz de Custos):** Sintonia do threshold em `mlp.predict_proba(X_test)[:, 1]` para minimizar a função de custo real de negócio: `Custo Total = (Falso Negativo * R$ 1.500) + (Falso Positivo * R$ 100)`.

---

## 7. MLOps: Deploy e Monitoramento
- **Estratégia de Implantação:** Microsserviço de inferência em tempo real via API REST (FastAPI/Docker) com tempo de resposta < 30ms.
- **Monitoramento Contínuo:**
  - *Data Drift:* Monitorar mudanças na distribuição de valores e frequências de transação por hora.
  - *Concept Drift:* Avaliação contínua semanal das fraudes confirmadas via contestação de chargeback para recalibração do modelo.
