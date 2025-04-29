# TelecomPlus – Relatório Técnico Completo

> Case de previsão de churn • Inteli Academy  
> Autor: Mateus Pereira • Data: 29 abr 2025

## Sumário
1. [Estrutura do Repositório](#estrutura-do-repositório)  
2. [Como Reproduzir](#como-reproduzir)  
3. [Metodologia & Etapas](#metodologia--etapas)  
4. [Exploração de Dados](#exploração-de-dados)  
5. [Pré-processamento](#pré-processamento)  
6. [Insights & Hipóteses](#insights--hipóteses)  
7. [Modelagem](#modelagem)  
8. [Avaliação Final](#avaliação-final)  
9. [Conclusões & Recomendações](#conclusões--recomendações)  

## Estrutura do Repositório

- `case.ipynb`: Notebook Jupyter com todo o processo de análise, modelagem e previsão
- `dados_clientes.csv`: Dataset de treino (98.868 registros após limpeza)
- `desafio.csv`: Dataset de teste (5.000 registros)
- `resultado_mateus_beppler_pereira.csv`: Arquivo de submissão com previsões
- `README.md`: Este documento com a explicação detalhada do projeto

## Como Reproduzir

1. Instale as dependências:
   ```
   pandas>=1.3.5
   numpy>=1.20.3
   matplotlib>=3.5.1
   seaborn>=0.11.2
   scikit-learn>=1.0.2
   ```

2. Execute o notebook `case.ipynb` na ordem das células
3. O arquivo `resultado_mateus_beppler_pereira.csv` será gerado automaticamente

## Metodologia & Etapas

O projeto foi desenvolvido seguindo estas etapas principais:

1. **Exploração inicial**: Análise da estrutura de dados, estatísticas, valores ausentes
2. **Pré-processamento**: Limpeza, feature engineering, transformação de variáveis
3. **Geração de insights**: Formulação e validação de hipóteses sobre fatores de churn
4. **Modelagem**: Experimentação com 3 algoritmos diferentes
5. **Avaliação final**: Seleção do modelo, análise de importância de features
6. **Previsão e submissão**: Aplicação do modelo no dataset de teste

Em cada etapa, aplicamos técnicas específicas e documentamos as decisões tomadas, com base tanto nos dados quanto no contexto de negócio.

## Exploração de Dados

### Dimensões e Estrutura

* **Dataset treino**: 98.872 × 18 (após remoção de 4 linhas com idades negativas)
* **Dataset teste**: 5.000 × 17
* **18 variáveis originais**, incluindo:
  * Identificador único (`id_cliente`)
  * Dados demográficos (`idade`, `genero`, `estado_civil`, `renda_faixa`)
  * Dados do contrato (`tipo_contrato`, `tempo_como_cliente`, `forma_pagamento`)
  * Comportamento de uso (`produtos_assinados`, `valor_mensal`, `total_gasto`) 
  * Dados de interação (`suporte_contatado`, `chamados_abertos`, `reclamacoes`)
  * Comportamento financeiro (`atrasos_pagamento`)
  * Alvo: `churn` (0=permanece, 1=cancela)

### Qualidade dos Dados

* **Valores ausentes**: Não foram encontrados (0%)
* **Duplicatas**: 0 em `id_cliente`
* **Inconsistências**: 
  * 4 registros com `idade` negativa (removidos no pré-processamento)
  * Demais colunas com valores dentro de faixas esperadas

### Distribuição da Variável-Alvo

* **Permanece (0)**: 66,3% dos clientes
* **Cancela (1)**: 33,7% dos clientes
* **Observação**: Desbalanceamento moderado, tratável com técnicas de balanceamento de classes

### Correlações Iniciais

Correlações mais fortes com `churn`:

* **`tempo_como_cliente`**: -0,33 (correlação negativa forte)
* **`total_gasto`**: -0,20 (correlação negativa moderada)
* **`atrasos_pagamento`**: +0,06 (correlação positiva fraca)
* **`suporte_contatado`**: +0,05 (correlação positiva fraca)

Interpretação preliminar: **Clientes mais novos, com menor gasto acumulado e mais atrasos de pagamento têm maior propensão a cancelar.**

## Pré-processamento

### Limpeza de Dados

* **Remoção de outliers**: 4 registros com `idade` < 0 (valores impossíveis)
* **Dados inconsistentes**: Não houve necessidade de outras remoções

### Feature Engineering

* **Extração de produtos**: 
  * Conversão da string `produtos_assinados` em listas Python
  * Criação de nova coluna `num_produtos` (contagem)
  * Criação de 6 features binárias (`has_a`, `has_b`, etc.) para os produtos mais comuns
  * Top-6 produtos identificados: Produtos B, C, E, D, F e A

* **Variáveis numéricas**: 14 após engenharia de features
* **Variáveis categóricas**: 5 originais transformadas em features binárias
* **Variáveis binárias**: 6 (flags de produtos)

### Pipeline de Transformação

Para garantir reprodutibilidade e evitar data leakage, criamos um pipeline completo de transformação:

* **Tratamento de variáveis numéricas**:
  * Imputação de valores faltantes com mediana (não necessário, mas incluído para robustez)
  * Normalização com `StandardScaler`

* **Tratamento de variáveis categóricas**:
  * Imputação com valor mais frequente
  * Codificação One-Hot com `OneHotEncoder`

* **Dimensionalidade final**: 26 features após transformações

## Insights & Hipóteses

Formulamos e testamos 4 hipóteses principais para entender os drivers de churn:

### Hipótese 1: Contratos mensais geram mais churn
* **Resultado**: ✅ Confirmada
* **Evidência**: Taxa de churn em contrato **Mensal** = 34,6% vs **Anual** = 31,6%
* **Implicações para o negócio**: 
  * Contratos mensais oferecem maior flexibilidade ao cliente, mas aumentam o risco de cancelamento
  * Estratégia: Oferecer incentivos para migração para planos anuais

### Hipótese 2: Atrasos de pagamento elevam a taxa de churn
* **Resultado**: ✅ Confirmada
* **Evidência**: Crescimento progressivo da taxa de churn com número de atrasos:
  * 0 atrasos: 30,0%
  * 1 atraso: 31,3% 
  * 2-3 atrasos: 34,6%
  * >3 atrasos: 37,8%
* **Implicações para o negócio**:
  * Clientes com problemas de pagamento recorrentes têm maior chance de cancelar
  * Estratégia: Implementar sistema de alertas para atrasos e facilitar opções de pagamento

### Hipótese 3: Clientes com menos de 12 meses cancelam mais
* **Resultado**: ✅ Fortemente confirmada
* **Evidência**: Queda significativa da taxa de churn com o tempo:
  * ≤ 12 meses: 68,4% (!!!)
  * 13-24 meses: 58,1%
  * 25-60 meses: 35,4% 
  * >60 meses: 20,6%
* **Implicações para o negócio**:
  * O primeiro ano é crítico para a retenção de clientes
  * Estratégia: Melhorar onboarding e acompanhamento no primeiro ano

### Hipótese 4: Reclamações e suporte são sinais de churn
* **Resultado**: ✅ Parcialmente confirmada
* **Evidência**: Grupo "Alto contato" (≥2 reclamações e ≥3 contatos) tem 38,5% de churn vs 33,6% no geral
* **Implicações para o negócio**:
  * Cliente que reclama mais tem maior risco, mas não é um sinal determinante
  * Estratégia: Melhorar solução no primeiro contato e follow-up após reclamações

### Síntese de Insights

Os principais drivers de churn identificados são:

1. **Temporalidade** - Risco cai drasticamente com a antiguidade do cliente
2. **Tipo de contrato** - Planos mensais têm maior rotatividade
3. **Problemas financeiros** - Atrasos são indicadores de propensão a cancelar
4. **Insatisfação** - Contatos frequentes com suporte aumentam moderadamente o churn

## Modelagem

### Conjunto de Dados & Split

* **Train-validation split**: 80%-20%, estratificado por `churn`
* **Balanceamento**: Aplicado via `class_weight='balanced'` nos modelos

### Modelos Testados

#### Modelo 1: Regressão Logística (baseline)
* **Hiperparâmetros**: solver='lbfgs', class_weight='balanced', penalty='l2'
* **Performance**: 
  * ROC-AUC: 0,7051
  * F1-score: 0,5649
  * Accuracy: 0,65 (abaixo do requisito mínimo)

#### Modelo 2: Gradient Boosting
* **Hiperparâmetros**: learning_rate=0.05, max_depth=6, class_weight balanceado
* **Performance**:
  * ROC-AUC: 0,7031
  * F1-score: 0,5625
  * Accuracy: 0,71
* **Observação**: Variação alta entre folds, risco de overfitting

#### Modelo 3: Random Forest (selecionado)
* **Hiperparâmetros otimizados via RandomizedSearchCV**:
  * n_estimators=700 (número de árvores)
  * max_depth=5
  * min_samples_split=4
  * min_samples_leaf=2
  * max_features=None
* **Performance**:
  * ROC-AUC: 0,7018
  * F1-score: 0,5620
  * Accuracy: 0,70 (atende requisito mínimo)

### Critério de Seleção

O **Random Forest** foi escolhido como modelo final porque:

1. **Atende o requisito mínimo** de accuracy ≥ 70%
2. Mantém métricas globais (AUC, F1) similares aos outros modelos
3. Oferece maior robustez e evita overfitting (Gradient Boosting mostrou instabilidade)
4. Permite fácil interpretação via importância de features

## Avaliação Final

### Performance do Random Forest

### Feature Importance (Top-10)

1. `tempo_como_cliente` - **92,2%** (dominante)
2. `atrasos_pagamento` - **3,3%**
3. `suporte_contatado` - **2,6%**
4. `total_gasto` - **0,4%**
5. `reclamacoes` - **0,3%**
6. `tempo_medio_atendimento` - **0,3%**
7. `tipo_contrato_Mensal` - **0,2%**
8. `idade` - **0,1%**
9. `valor_mensal` - **0,1%**
10. `tipo_contrato_Anual` - **0,1%**

Interpretação: O modelo confirmou quantitativamente as hipóteses testadas, com o tempo como cliente sendo o fator predominante de previsão.

## Conclusões & Recomendações

### Principais Conclusões

1. **O fator tempo é determinante**: Clientes com menos de 12 meses têm probabilidade 3x maior de cancelar (68% vs 21% após 5 anos)
2. **Contratos anuais retêm melhor**: 10% menos churn vs contratos mensais
3. **Inadimplência é sinal de alerta**: Cada atraso adicional aumenta progressivamente a chance de cancelamento
4. **Suporte é importante, mas secundário**: Cliente que reclama tem risco aumentado, mas não é o driver principal

### Recomendações de Negócio

1. **Programa de retenção no primeiro ano**:
   * Acompanhamento ativo nos primeiros 3, 6 e 12 meses
   * Questionários de satisfação rápidos e frequentes
   * Oferta de valor especial ao completar 12 meses

2. **Incentivo a contratos anuais**:
   * Descontos progressivos para contratos mais longos
   * Benefícios exclusivos para planos anuais
   * Descontos na renovação antecipada

3. **Monitoramento de pagamentos**:
   * Sistema de alerta após primeiro atraso
   * Contato proativo para entender motivo do atraso
   * Flexibilização de datas de pagamento

4. **Melhoria do suporte**:
   * Priorização de clientes com múltiplos chamados
   * Follow-up posterior para verificar satisfação
   * Empoderamento do primeiro nível para resolver problemas