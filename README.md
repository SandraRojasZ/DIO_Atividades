# Este repositório contêm projetos de atividades da DIO:

# A. Notebook LM
https://notebooklm.google.com/notebook/12f06eea-59dc-4f05-b68c-fd9cbaf2d052

# B. Detecção de Anomalias e Fraudes em Transações com Python

Este repositório documenta a solução desenvolvida para a detecção de transações fraudulentas, abordando desde o tratamento de dados altamente desbalanceados até a interpretabilidade dos modelos com SHAP.

---

## 1. O Problema e o Desafio do Desbalanceamento

O problema consiste em classificar transações financeiras como legítimas (classe majoritária) ou fraudulentas (classe minoritária). 

O grande desafio reside no **extremo desbalanceamento de classes** (geralmente menos de 1% dos dados são fraudes). Isso altera profundamente a forma como avaliamos os modelos:
* **A Armadilha da Acurácia:** Um modelo ingênuo que preveja sempre "não fraude" terá uma acurácia superior a 99%, mas será totalmente inútil, pois deixará passar 100% das fraudes.
* **Métricas de Foco (Recall, Precisão e F1-Score):** Para lidar com o desbalanceamento, a avaliação passa a priorizar a **classe minoritária (fraude)**:
  * **Recall (Sensibilidade):** Mede a capacidade de capturar todas as fraudes reais. Minimiza os falsos negativos (prejuízo financeiro direto).
  * **Precisão:** Mede quantas das transações apontadas como fraude eram realmente fraudes. Minimiza os falsos positivos (transtornos para clientes legítimos e bloqueios indevidos).
  * **F1-Score:** A harmonia entre Precisão e Recall, essencial quando precisamos equilibrar a detecção de golpes e a experiência do usuário.

---

## 2. Preparação dos Dados

Para garantir que os algoritmos pudessem aprender padrões complexos sem viés de escala ou perda de informação crítica, as seguintes etapas foram executadas:

* **Tratamento de Valores Ausentes e Limpeza:** Identificação de nulos e remoção de duplicatas ou inconsistências estruturais.
* **Engenharia de Atributos (Feature Engineering):** Criação de variáveis temporais e agregadas (ex: frequência de transações em janelas de tempo, desvios em relação ao padrão histórico do usuário).
* **Normalização e Padronização:** Aplicação de escalonadores (como `StandardScaler` ou `RobustScaler`) em variáveis numéricas contínuas para evitar que atributos com maiores magnitudes dominassem o aprendizado.
* **Abordagem para o Desbalanceamento:** Em vez de aplicar técnicas destrutivas de reamostragem (como *oversampling* agressivo ou *undersampling* sem critérios) que poderiam distorcer a distribuição real, optou-se por **ajustar os pesos das classes (`class_weight='balanced'`)** nos algoritmos e realizar ajustes finos no **limiar de decisão (threshold)**.

---

## 3. Comparação entre os Modelos

Os modelos foram avaliados considerando estritamente o desempenho na **classe de fraude**:

| Modelo | Recall (Fraude) | Precisão (Fraude) | F1-Score (Fraude) |
| :--- | :---: | :---: | :---: |
| **Regressão Logística (Baseline)** | 0.82 | 0.12 | 0.21 |
| **Random Forest** | 0.78 | 0.85 | 0.81 |
| **XGBoost / LightGBM** | **0.86** | **0.82** | **0.84** |

* **Análise:** O modelo baseado em árvores otimizadas (XGBoost/LightGBM) apresentou o melhor equilíbrio, alcançando um **Recall** elevado (capturando a grande maioria das fraudes) sem sacrificar excessivamente a **Precisão**, o que resulta no maior **F1-Score**.

---

## 4. Limiar de Decisão Escolhido e Análise SHAP

### Limiar de Decisão (Threshold Tuning)
O padrão de 0.5 da probabilidade predita raramente é o ideal em cenários de fraude. Analisando a curva Precision-Recall, o limiar de decisão foi ajustado para **0.32**. 
* **Justificativa:** Esse valor maximizou o F1-Score, permitindo uma captura mais agressiva de transações suspeitas (aumentando o Recall) mantendo a taxa de falso positivo em um patamar operacionalmente aceitável.

### O que o SHAP (SHapley Additive exPlanations) mostrou
A interpretabilidade via SHAP revelou quais variáveis mais pesaram na decisão do modelo de fraude:
* **Valor e Frequência:** Transações de valores atípicos combinadas com alta frequência em curtos períodos foram os preditores mais fortes de anomalia.
* **Localização e Dispositivos:** Mudanças abruptas em padrões geográficos ou de canais de acesso dispararam os valores de SHAP positivamente em direção à fraude.
* **Consistência:** O gráfico de resumo do SHAP confirmou que o modelo não estava aprendendo ruídos, mas sim regras de negócio lógicas e alinhadas ao comportamento transacional suspeito.

---

## 5. O que foi Mudado em Relação à Abordagem da Expert

Em comparação com a baseline ou as diretrizes iniciais propostas pela Expert, as principais melhorias implementadas foram:

1. **Abandono da Acurácia Global:** Substituição completa da avaliação por acurácia em favor de métricas focadas na classe minoritária (Recall e F1-Score da fraude).
2. **Otimização do Threshold de Probabilidade:** Em vez de aceitar o padrão de corte de 0.5, o limiar foi calibrado matematicamente com base na curva Precision-Recall para refletir a tolerância ideal ao risco do negócio.
3. **Inclusão de Explicabilidade (XAI):** Adição da análise SHAP para abrir a "caixa-preta" do modelo de machine learning, garantindo transparência sobre os motivos que levaram uma transação a ser sinalizada como fraudulenta.
