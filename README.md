# Previsão e Detecção de Anomalias no Consumo de Energia em Residências Inteligentes

## 1. Descrição do Problema

Com o crescente uso de dispositivos eletrônicos e sistemas de climatização em residências, monitorar e prever o consumo de energia elétrica tornou-se essencial para otimizar custos e promover a sustentabilidade. Este projeto visa desenvolver modelos de Machine Learning capazes de:

- **Prever** o consumo de energia elétrica (`energy_consumption_kWh`) em residências inteligentes com base em variáveis como temperatura, status de ocupação, uso de eletrodomésticos e características temporais.
  
- **Detectar** anomalias no consumo de energia que possam indicar comportamentos atípicos ou falhas nos sistemas.

**Como faria isso?**  
Esses modelos estariam no firmware de plugs de tomada inteligentes, que monitoram continuamente o consumo de kWh e outras variáveis. Isso permite tanto prever o consumo elétrico para automatizar certas ações quanto identificar anomalias. Com base nas anomalias detectadas, o cliente recebe uma notificação via aplicativo que indica que algo está errado. Através da notificação, o cliente pode:

- Verificar o dispositivo em questão para identificar a causa.
- Responder ao sistema.
- Desligar o fornecimento de energia remotamente, caso necessário.

As anomalias podem variar desde um simples problema até algo mais sério, como um equipamento danificado.

## 2. Metodologia

### 2.1. Coleta e Pré-processamento dos Dados



**Carregamento dos Dados:**  
Utilizamos um dataset de uso de energia em residências inteligentes, contendo informações como consumo energético, configurações de temperatura, status de ocupação, uso de eletrodomésticos e dados temporais. https://www.kaggle.com/datasets/arnavsmayan/smart-home-energy-usage-dataset?resource=download

**Limpeza dos Dados:**

- Remoção de registros com consumo de energia negativo.
- Conversão da coluna `occupancy_status` para binário (1 = Occupied, 0 = Unoccupied).

**Engenharia de Features Temporais:**

- Extração de informações como ano, mês, dia, dia da semana e hora a partir da coluna `timestamp`.
- Criação de indicadores adicionais, como `peak_hour` (horário de pico das 18h às 21h).

**Agregação dos Dados:**

- Agregamos os dados diariamente por `home_id`, somando o consumo de energia e calculando médias de temperatura, além de outras métricas relevantes.

**Codificação de Variáveis Categóricas:**

- Aplicamos **One-Hot Encoding** nas colunas categóricas (`season`, `day_of_week`, `appliance`) para convertê-las em formato numérico.

**Tratamento de Valores Ausentes e Escalonamento:**

- Utilizamos `SimpleImputer` para substituir valores faltantes pela média.
- Aplicamos `StandardScaler` para normalizar as features numéricas.

### 2.2. Modelagem

#### 2.2.1. Random Forest Regressor

**Objetivo:**  
Prever o consumo de energia elétrica (`energy_consumption_kWh`).

**Configuração:**  
Utilizamos um modelo de Random Forest com:

- 100 árvores (`n_estimators=100`)
- `random_state=42` para reprodutibilidade
- `n_jobs=-1` para paralelização

**Avaliação:**  
As métricas utilizadas foram:

- **MAE (Mean Absolute Error)**
- **RMSE (Root Mean Squared Error)**
- **R² (Coeficiente de Determinação)**

#### 2.2.2. Autoencoder

**Objetivo:**  
Detectar anomalias no consumo de energia.

**Arquitetura:**  
Construímos um Autoencoder com:

- Uma camada de codificação de 8 neurônios
- Uma camada de decodificação correspondente
- Regularização L1 para evitar overfitting

**Treinamento:**  
Treinamos o modelo com:

- 50 épocas
- `batch_size` de 32
- Monitoramento da perda de validação

**Detecção de Anomalias:**  
Calculamos o erro de reconstrução e definimos um limiar baseado no percentil 95 para identificar anomalias.

## 3. Resultados Obtidos

### 3.1. Random Forest Regressor

- **MAE (Mean Absolute Error):** 1.23 kWh  
  *Interpretação:* Em média, as previsões do modelo têm um erro absoluto de aproximadamente 1.23 kWh em relação ao consumo real.

- **RMSE (Root Mean Squared Error):** 1.42 kWh  
  *Interpretação:* O modelo comete erros ligeiramente maiores em alguns casos, já que o RMSE penaliza mais erros altos.

- **R² (Coeficiente de Determinação):** -0.0186  
  *Interpretação:* O valor negativo indica que o modelo performa pior do que uma abordagem simples baseada na média dos valores reais.

### 3.2. Autoencoder

**Perda de Treino e Validação:**  
Observamos uma redução consistente na perda de treino e validação ao longo das épocas, indicando que o modelo está aprendendo a reconstruir os dados.

**Detecção de Anomalias:**

- **Limiar de Anomalia:** Definido no percentil 95 dos erros de reconstrução.
- **Anomalias Detectadas:** Foram identificadas diversas anomalias no conjunto de teste, indicando padrões atípicos no consumo de energia.

## 4. Conclusões

O projeto demonstrou o processo de previsão e detecção de anomalias no consumo de energia em residências inteligentes utilizando **Random Forest Regressor** e **Autoencoder**. Os principais pontos observados são:

- **Random Forest Regressor:**  
  Apesar de ser um modelo robusto para dados tabulares, os resultados obtidos (MAE: 1.23 kWh, RMSE: 1.42 kWh, R²: -0.0186) indicam que o modelo atual não está capturando eficazmente os padrões de consumo de energia. O R² negativo sugere que o modelo é menos eficaz do que uma previsão baseada na média dos valores reais, possivelmente devido à necessidade de ajustes nos hiperparâmetros ou inclusão de features mais relevantes.

- **Autoencoder:**  
  O modelo conseguiu aprender a reconstruir os dados e identificar anomalias com base no erro de reconstrução. A definição de um limiar no percentil 95 permitiu detectar comportamentos atípicos no consumo energético, demonstrando a eficácia do Autoencoder para a tarefa de detecção de anomalias.
