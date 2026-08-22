# Arquitetura e Planejamento: Projeto Lumina - Diagnóstico e Portabilidade Inteligente de Crédito

## 1. Visão Geral do Problema e Objetivo

### O Problema
A Lumina é uma fincare brasileira em estágio de validação de mercado que oferece saúde financeira como benefício corporativo para empresas contratantes que pagam uma mensalidade por colaborador (modelo B2B2C). O diferencial do produto é a abordagem Rate First, Payoff Second: diagnosticar contratos de crédito ativos (empréstimo consignado, financiamento de veículos e crédito imobiliário) com taxas acima do mercado e orientar o trabalhador CLT a buscar a portabilidade antes de estruturar o plano de quitação de dívidas.

O principal gargalo operacional do produto está na assertividade da recomendação: quando o usuário cadastra seus contratos, a Lumina precisa estimar se uma tentativa de portabilidade resultará em economia real e aceitação no mercado. Errar essa previsão tem custos fortemente assimétricos: 1. Falso positivo (recomendar portabilidade que o mercado não aceita em condições melhores): o trabalhador tenta a portabilidade, sofre recusa ou não obtém ganho financeiro real, frustra-se e perde a confiança no diagnóstico da Lumina, que é o coração do produto. 2. Falso negativo (deixar de recomendar uma portabilidade viável): o trabalhador segue pagando juros abusivos, e a Lumina falha na entrega de valor prometida à empresa contratante do benefício.

Diferente de um banco tradicional, onde o custo do erro se traduz em provisão para devedores duvidosos (inadimplência/calote), na Lumina o custo do erro reside na perda de confiança do usuário e no churn do benefício corporativo.

### Objetivo da IA
Classificar e estimar a probabilidade de um contrato de crédito ativo resultar em portabilidade aceita e economicamente vantajosa, calibrando a recomendação para maximizar a precisão e gerar justificativas claras e transparentes para o trabalhador.

### Métrica de Sucesso (Negócio)
Como a Lumina está em estágio de validação e não possui histórico volumétrico para estabelecer baselines consolidados, as metas são declaradas explicitamente como hipóteses iniciais a validar:
- **Hipótese 1:** Taxa de sucesso das portabilidades recomendadas superior a 80% (alta precisão para preservar a confiança no diagnóstico).
- **Hipótese 2:** Churn anual de empresas contratantes do benefício corporativo abaixo de 10%, sustentado pela percepção de valor e economia real gerada para os colaboradores.
- **Hipótese 3:** Engajamento ativo no diagnóstico superior a 60% dos colaboradores cadastrados nas empresas clientes.

## 2. Coleta e Preparação de Dados

### Fase Atual (MVP)
- **Fontes de dados:**
  - Cadastro manual de contratos pelo próprio usuário no aplicativo web/mobile (tipo de contrato, taxa de juros contratada, saldo devedor aproximado, prazo restante e renda mensal informada), com validação de faixas plausíveis por modalidade na interface.
  - Tabelas públicas de taxas médias de juros por modalidade disponibilizadas pelo Banco Central do Brasil em dados abertos (sem SCR e sem integrações de Open Finance).
- **Qualidade dos dados:**
  - Validação de faixas plausíveis na entrada do formulário (exemplo: limites normativos para consignado CLT, prazos máximos para veículos e limites de comprometimento de renda).
  - Tratamento de campos nulos com obrigatoriedade na coleta e verificações de consistência (sanity checks) na camada de aplicação.
- **Engenharia de features:**
  - Spread de taxa em relação à média de mercado do BACEN (taxa_atual_am - taxa_referencia_mercado).
  - Estimativa simplificada de comprometimento de renda (parcela estimada dividida pela renda declarada).
  - Termo de interação central entre spread, prazo restante e saldo devedor para representar a diluição de atritos e custos operacionais de transação.
  - One-hot encoding para a variável categórica tipo_contrato (consignado, veiculo, imobiliario) com drop_first=True.

### Visão de Escala
- Integração com Open Finance Brasil via APIs reguladas para extração automática e padronizada de saldos, taxas efetivas (CET) e cronogramas de amortização.
- Integração direta com sistemas de folha de pagamento e RH (ERP/HRIS) para captura automática de margem consignável, renda líquida e tempo de vínculo empregatício.
- Módulo de OCR para leitura de contratos e extratos bancários (CCB).
- Evoluções condicionadas a volume real de usuários e tração do negócio.

## 3. Estratégia de Bases e Separação de Dados

### Fase Atual (MVP)
- **Armazenamento:**
  - Banco de dados relacional PostgreSQL gerenciado no Supabase, aproveitando a mesma infraestrutura do produto.
  - Isolamento e anonimização de identificadores pessoais (CPF, nome, e-mail) para a camada de modelagem e análise, em estrita conformidade com a LGPD.
- **Metodologia de split:**
  - Divisão estratificada (75% treino e 25% teste) com preservação da proporção de portabilidades vantajosas em cada partição.
- **Prevenção de vazamento (Data Leakage):**
  - Ajuste (fit) de todos os transformadores (StandardScaler e encoders) estritamente no conjunto de treino.
  - Taxas de mercado públicas associadas ao contrato fixadas na data de simulação, sem uso de indicadores futuros.

### Visão de Escala
- Estruturação de Data Lakehouse (S3 ou GCS com Delta Lake / Parquet) com camadas Raw, Bronze, Silver e Gold.
- Feature Store centralizada para reuso de atributos em tempo real e batch.
- Validação temporal automatizada por safras de contratos (Out-of-Time Split) para acompanhar ciclos macroeconômicos e variações da taxa Selic.
- Evolução condicionada ao aumento volumétrico de dados históricos.

## 4. Seleção de Algoritmos

### Modelo Baseline
- **Regressão Logística:** modelo linear clássico, rápido de treinar, estável e com interpretabilidade direta através de seus coeficientes. Serve como referência obrigatória de desempenho e transparência.

### Modelos Candidatos Testados
- **Rede Neural Perceptron Multicamadas (MLPClassifier):** arquitetura feedforward com duas camadas ocultas (16 e 8 neurônios) e ativação ReLU. Testada no notebook 14 para avaliar a capacidade de capturar interações não lineares entre taxas, saldos e prazos sem engenharia manual prévia.
- **Resultados obtidos na validação multi-seed (seeds 42, 7 e 123):** o MLP obteve AUC-ROC médio de 0,883 +- 0,010 contra 0,861 +- 0,011 da Regressão Logística. Na seed 123, os modelos empataram tecnicamente (0,876 vs 0,873), indicando que o ganho é modesto e sensível à partição amostral.

### Modelos Candidatos Futuros (Não Testados)
- **LightGBM / XGBoost:** algoritmos de Gradient Boosting baseados em árvores, planejados para avaliação futura caso o volume de dados e o número de variáveis cadastrais aumentem significativamente.

### Decisão Técnica Registrada
Enquanto a vantagem de performance de modelos não lineares (como MLP ou Gradient Boosting) for modesta e não consistente entre sementes, a Regressão Logística calibrada permanece como a escolha oficial de produção. Para a Lumina, a explicabilidade da recomendação em linguagem simples para o trabalhador é um requisito central de produto, superando ganhos marginais de métricas em modelos de caixa-preta.

## 5. Estratégia de Treinamento e Otimização

### Função de Custo (Loss Function)
- **Log-Loss (Binary Cross-Entropy):** minimização do erro de entropia cruzada para predição de probabilidades.
- **Calibração de Probabilidades Pós-Treino:** aplicação de Platt Scaling (regressão logística sobre as saídas) ou Regressão Isotônica para garantir que uma probabilidade estimada de 80% corresponda efetivamente a 80% de chance de portabilidade bem-sucedida.

### Otimização de Hiperparâmetros
- **Busca em Grade (GridSearchCV) ou Random Search:** sintonia de regularização L2 (parâmetro C na Regressão Logística) e arquitetura/taxa de aprendizado no MLP com validação cruzada estratificada em 5 folds.

## 6. Testes, Validação e Métricas

### Métricas de Avaliação Técnica
- **AUC-ROC:** métrica principal para avaliar a capacidade de ordenação de risco e separação entre contratos vantajosos e desvantajosos.
- **Brier Score:** métrica de calibração de probabilidade para avaliar a precisão das probabilidades estimadas (quanto mais próximo de 0, melhor a calibração).
- **Precisão e Recall no Limiar Operacional:** monitoramento do trade-off entre capturar oportunidades e evitar recomendações erradas.

### Matriz de Confusão e Impacto no Negócio

| Classificação do Modelo | Realidade de Mercado: Inviável | Realidade de Mercado: Vantajosa |
| :--- | :--- | :--- |
| **Predição: Não Recomendar (0)** | **Verdadeiro Negativo:** Usuário é direcionado para plano de quitação sem frustração. | **Falso Negativo:** Oportunidade perdida; trabalhador segue com juros altos (Impacto Moderado). |
| **Predição: Recomendar (1)** | **Falso Positivo:** Frustração do usuário, descrédito no diagnóstico Lumina, risco de churn B2B (Impacto Crítico). | **Verdadeiro Positivo:** Economia real gerada, validação do produto e recomendação espontânea. |

**Ajuste de Limiar Operacional:** O ponto de corte de probabilidade é calibrado para priorizar alta precisão, evitando disparar recomendações de portabilidade duvidosas.

### Explicabilidade Nativa (Requisito de Produto)
O modelo precisa alimentar mensagens diretas no aplicativo que expliquem o motivo do diagnóstico, tais como:
> "Identificamos que sua taxa atual de 2,2% a.m. está 0,75 ponto percentual acima da taxa média de mercado para crédito consignado. Como ainda restam 42 parcelas sobre um saldo de R$ 18.000, a portabilidade pode economizar cerca de R$ 2.900 ao longo do contrato."

### Testes de Robustez e Generalização
- Validação multi-seed com pelo menos 3 sementes (42, 7 e 123) em todas as comparações de modelos para verificar consistência e desvio-padrão dos resultados.

## 7. MLOps: Deploy e Monitoramento

### Fase Atual (MVP)
- **Estratégia de implantação:**
  - Arquitetura serverless enxuta e alinhada à stack atual da Lumina (Supabase, Netlify Functions e Claude API).
  - O modelo treinado e calibrado (Regressão Logística) é serializado em formato leve e executado diretamente dentro de uma Netlify Function.
  - Sem necessidade de containers Docker dedicados, servidores permanentes ou orquestradores de infraestrutura (como Kubernetes).
- **Monitoramento contínuo:**
  - Processo manual mensal de revisão: cruzamento entre os diagnósticos emitidos no app e os desfechos reportados pelos próprios usuários e pelas empresas parceiras.
  - Acompanhamento da taxa de conversão real e das razões de recusa reportadas.
- **Gatilho de retreino:**
  - Qualitativo e orientado a feedback: reavaliação dos parâmetros do modelo quando houver queda perceptível na taxa de sucesso reportada ou quando o Banco Central publicar alterações relevantes nas tabelas de taxas médias.

### Visão de Escala
- Microserviço dedicado de inferência em tempo real utilizando FastAPI e empacotamento em containers gerenciados.
- Pipelines de processamento em lote para reanálise periódica automatizada de toda a carteira de contratos da base.
- Monitoramento contínuo de Data Drift e Concept Drift com ferramentas automatizadas (como Evidently AI).
- Pipeline de retreinamento automatizado com testes de regressão antes do deploy contínuo (CI/CD de ML).
- Evoluções condicionadas a tração de mercado e volume real de contratos analisados.
