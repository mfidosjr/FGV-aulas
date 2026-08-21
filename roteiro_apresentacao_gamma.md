# Roteiro de Apresentação Executiva: Detecção Inteligente de Fraudes Financeiras

---

# Slide 1: O Fim da Fraude Silenciosa — Protegendo a Receita e a Experiência do Cliente
**Subtítulo:** Como a Inteligência Artificial Transforma a Prevenção de Perdas Financeiras em uma Vantagem Competitiva.

- **O Desafio Atual:** Instituições financeiras perdem milhões de reais anualmente com fraudes transacionais, enquanto arcam com altos custos operacionais de suporte ao cliente ao tentar bloquear transações suspeitas de forma manual ou por regras rígidas.
- **A Limitação Atual:** Regras de decisão estáticas e modelos lineares tradicionais não conseguem acompanhar a sofisticação dos fraudadores, gerando um dilema entre ter um alto índice de fraudes não detectadas ou causar atrito ao bloquear indevidamente cartões de clientes legítimos (Falsos Positivos).
- **A Visão com Inteligência Artificial:** Uma arquitetura de Redes Neurais (*Multi-Layer Perceptron*) adaptada para dados desbalanceados, capaz de identificar padrões não-lineares complexos em milissegundos, reduzindo perdas por fraude e eliminando o atrito no atendimento.

---

# Slide 2: Por Dentro da Solução — Como Nossa Rede Neural Identifica Padrões Ocultos
**Subtítulo:** Tecnologia de Ponta Aplicada à Análise Comportamental em Tempo Real sem Complicação.

- **Nossos Dados:** Utilização de dados transacionais já existentes nos gateways de pagamento e sistemas internos (valor, horário, histórico de frequência em 24h, geolocalização e novos dispositivos).
- **A Abordagem Inteligente:** Treinamento especializado com **Oversampling** da classe minoritária, permitindo que a Rede Neural aprenda combinações triplas complexas (ex.: compra de alto valor + dispositivo novo + horário de madrugada) mantendo estabilidade total e 0% de colapso de modelo.
- **Integração Fluida:** O modelo atua como um microsserviço de inferência contínua via API REST (latência < 30ms), scoreando transações em tempo real de forma transparente para a equipe de fraude e para o cliente final.

---

# Slide 3: Impacto de Negócio e Roadmap de Entrega — Rumo a uma Operação Escalável
**Subtítulo:** Retorno Financeiro Quantificável, Validação Robusta e Implantação Segura.

- **Impacto Esperado:** Aumento de **+23,5% no F1-Score** e ganho de **+43% em Precisão** sobre o baseline tradicional, com taxa constante de captura de fraudes (*Recall* > 51%), otimizando a matriz de custos reais de negócio ($FN vs $FP).
- **Validação Segura:** Validação empírica contínua através de testes **multi-seed (seeds 42, 7 e 123)** comprovando consistência e ausência de instabilidade, seguida de fase inicial em *Shadow Mode* (operação em paralelo sem impacto no produção).
- **Próximos Passos (Roadmap):**
  1. *Fase 1 (Semanas 1-2):* Homologação da API de inferência e integração com o Data Lake.
  2. *Fase 2 (Semanas 3-4):* Teste piloto em *Shadow Mode* e ajuste fino do limiar de probabilidade (`predict_proba`).
  3. *Fase 3 (Mês 2):* Lançamento oficial em produção e monitoramento contínuo de *Data Drift*.
