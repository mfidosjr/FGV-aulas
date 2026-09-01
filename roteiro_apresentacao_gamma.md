# Roteiro de Apresentação Executiva: Detecção Inteligente de Fraudes Financeiras

**Apresentação Executiva (3 Slides)** | MBA em Inteligência Artificial para Negócios — FGV  
**Autor:** Renan Marchini Andrusiac Vieira  
**Tema:** O Fim da Fraude Silenciosa — Protegendo a Receita e a Experiência do Cliente

---

# Slide 1: O Fim da Fraude Silenciosa — Protegendo a Receita e a Experiência do Cliente
**Subtítulo:** Como a Inteligência Artificial Transforma a Prevenção de Perdas Financeiras em uma Vantagem Competitiva.

- **O Desafio Atual:** Instituições financeiras enfrentam prejuízos anuais expressivos com fraudes não detectadas (Falsos Negativos a R$ 1.500 por ocorrência), além de arcarem com altos custos operacionais de suporte e atrito ao bloquear indevidamente clientes legítimos (Falsos Positivos a R$ 100 por ocorrência).
- **A Limitação Atual:** Regras de decisão estáticas (SE-ENTÃO) e modelos lineares tradicionais não conseguem capturar interações não lineares complexas entre múltiplas variáveis, gerando regras ineficientes (F1 de 0,059) ou modelos lineares com excesso de falsos alarmes (1.400 FPs no teste).
- **A Visão com Inteligência Artificial:** Uma arquitetura de Redes Neurais (*Multi-Layer Perceptron*) com tratamento de desbalanceamento por Oversampling, capaz de identificar padrões ocultos em milissegundos (<30ms), equilibrando captura de fraudes e minimização drástica do atrito no atendimento.

---

# Slide 2: Por Dentro da Solução — Como Nossa Rede Neural Identifica Padrões Ocultos
**Subtítulo:** Tecnologia de Ponta Aplicada à Análise Comportamental em Tempo Real sem Complicação.

- **Nossos Dados:** Utilização de dados transacionais dos gateways de pagamento e sistemas de segurança (`valor_transacao`, `hora_do_dia`, `distancia_do_local_habitual_km`, `transacoes_ultimas_24h` e `dispositivo_novo`).
- **A Abordagem Inteligente:** Rede Neural MLP de 2 camadas ocultas `(16, 8)` com ativações ReLU e **Oversampling** da classe minoritária no treino, aprendendo combinações não lineares triplas e eliminando 100% dos colapsos de gradiente.
- **Integração Fluida e MLOps:** Microsserviço de inferência em tempo real via API REST (FastAPI/Docker/Kubernetes) com tempo de resposta **< 30ms (SLA p95)**, cache online Redis (<2ms) e governança com circuit breaker de fallback.

---

# Slide 3: Impacto de Negócio e Roadmap de Entrega — Rumo a uma Operação Escalável
**Subtítulo:** Retorno Financeiro Quantificável, Validação Robusta e Implantação Segura.

- **Impacto Técnico e Financeiro:**
  - **Métricas Multi-Seed Validadas (Seeds 42, 7, 123):** Ganho médio de **+43,8% em Precisão** ($0,138 \pm 0,019$ vs $0,096 \pm 0,002$) e **+30,7% em F1-Score** ($0,217 \pm 0,024$ vs $0,166 \pm 0,002$), com Recall de $51,4\% \pm 0,9\%$ (estabilidade comprovada).
  - **Viabilidade Financeira:** Economia projetada de **R$ 504.000,00/ano** em produção (1M transações/mês), gerando um **ROI de +323,5% no 1º ano** (Capex R$ 65k + Opex R$ 54k/ano) com **Payback de apenas 2,8 meses**.
- **Validação Segura:** Validação empírica multi-seed e calibração de limiar ótimo de decisão (threshold 0.80) minimizando o custo total de negócio para R$ 238.600,00 no conjunto de teste.
- **Próximos Passos (Roadmap de MLOps):**
  1. *Fase 1 (Semanas 1-2):* Homologação da API de inferência (<30ms) e integração contínua com o pipeline de dados.
  2. *Fase 2 (Semanas 3-4):* Piloto em *Shadow Mode* e ajuste fino de limiar de probabilidade.
  3. *Fase 3 (Mês 2+):* Rollout gradual (10% $\to$ 100%), monitoramento contínuo de Data Drift (PSI / KS-test), Concept Drift (feedback de chargeback) e retreino mensal programado.
