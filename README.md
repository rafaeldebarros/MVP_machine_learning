# ⚖️ MVP - Machine Learning & Analytics: Previsão de Desfechos de Recursos no STJ

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Data Science](https://img.shields.io/badge/Data%20Science-Machine%20Learning-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-MVP%20Conclu%C3%ADdo-success?style=for-the-badge)

## 📌 Visão Geral do Projeto

Este projeto apresenta um **MVP (Minimum Viable Product)** focado no desenvolvimento e avaliação de um modelo preditivo de classificação supervisionada. O objetivo principal é estimar a probabilidade de êxito de recursos jurídicos no **Superior Tribunal de Justiça (STJ)**, especificamente para decisões terminativas da *1ª Seção*, proferidas entre janeiro e maio de 2026. 

O modelo visa prever um desfecho binário:
* **Favorável** ao recorrente.
* **Desfavorável** ao recorrente.

---

## 💼 Contexto e Problema de Negócio

O crescente volume de processos nos Tribunais Superiores brasileiros gera uma demanda latente por ferramentas que auxiliem na gestão da **litigância estratégica**. A capacidade de antever o desfecho de recursos é crucial para advogados, defensores e entes públicos, permitindo:

* 🎯 Otimizar a alocação de recursos financeiros e humanos.
* 📉 Mitigar riscos financeiros processuais.
* 💡 Apoiar a tomada de decisão estratégica baseada em dados reais.

---

## 🎯 Objetivos do MVP

* Desenvolver um modelo preditivo de classificação supervisionada.
* Estimar a probabilidade de êxito de um recurso com base em seus atributos tangíveis.
* Validar se o **perfil do Ministro Relator** e o **assunto do processo** são preditores estatisticamente relevantes.
* Superar um *baseline* ingênuo de desempenho.

---

## 📊 Conjunto de Dados

Os dados utilizados provêm de duas fontes principais do ecossistema público de dados:

1.  **STJ (Superior Tribunal de Justiça):** Metadados e textos integrais de decisões terminativas da 1ª Seção. [dadosabertos.web.stj.jus.br](https://dadosabertos.web.stj.jus.br)
2.  **CNJ (Conselho Nacional de Justiça):** Dicionário de códigos de assuntos jurídicos para traduzir os códigos internos do STJ. [dpj.cnj.jus.br](https://dpj.cnj.jus.br/sgt/api/v1.0/assuntos.csv)

### ⚠️ Desafios Críticos dos Dados:
* **Valores Ausentes:** Alta taxa de nulos na coluna `texto_integral` (**82.31%**), além de ausências em `descricaoMonocratica` e `recurso` (tratados como informativos).
* **Desbalanceamento de Classes:** O desfecho 'Desfavorável' é a classe majoritária, exigindo tratamento e métricas especiais de avaliação.

---

## 🛠️ Metodologia e Pipeline

### 1. Carregamento e Limpeza
* Ingestão de metadados JSON e textos TXT mapeados no GitHub.
* Tradução dos códigos de assuntos do CNJ.
* Classificação do desfecho alvo (`Resultado_Estimado`) com base no campo textual `teor`.

### 2. Análise Exploratória de Dados (EDA)
* Mapeamento de valores faltantes e desbalanceamento de classes.
* Análise descritiva do comportamento dos Ministros e taxas de sucesso por assunto/classe processual.
* **Descoberta Importante:** Detecção de *data leakage* inicial provocado pelas colunas `descricaoMonocratica` e `texto_integral`.

### 3. Pré-processamento & Engenharia de Features
* 🛡️ **Remoção de Data Leakage:** Exclusão estratégica das colunas `descricaoMonocratica` e `texto_integral` para garantir que o modelo aprenda apenas com dados realisticamente disponíveis no momento do protocolo.
* **Tratamento de Nulos:** Preenchimento de nulos em `recurso` com `'Nao_Informado'` e em `assuntos_nomeados` com listas vazias.
* **Codificação Categórica:** Aplicação de `OneHotEncoder` para `tipoDocumento`, `NM_MINISTRO` e `recurso`.
* **Processamento de Texto:** Uso de `TfidfVectorizer` para `assuntos_nomeados` com remoção de *stopwords* em português.
* **Seleção de Features:** Aplicação de `SelectKBest` com teste Chi-Quadrado ($\chi^2$) para reduzir a dimensionalidade para as **1000 features** mais relevantes.

### 4. Modelagem e Avaliação
* **Divisão Treino/Teste:** Proporção 80/20 estratificada pela variável alvo.
* **Métrica Principal:** `F1-Macro` (escolhida especificamente devido ao desbalanceamento de classes).
* **Modelos Testados:** `DummyClassifier` (baseline), Regressão Logística, Decision Tree, Random Forest, LightGBM, CatBoost e XGBoost.
* **Mitigação de Desbalanceamento:** Uso dos parâmetros `class_weight='balanced'` ou `scale_pos_weight`.
* **Hiperparâmetros:** Ajustados via `GridSearchCV` e `RandomizedSearchCV`.

---

## 🚀 Resultados Principais

### 🔥 Desempenho dos Modelos (Pós-Remediação de Data Leakage)

| Modelo | F1-Macro (Teste) | Observações |
| :--- | :---: | :--- |
| **Random Forest Classifier** | **0.5754** | **Melhor desempenho geral em teste (robusto contra overfitting).** |
| XGBoost Classifier | 0.5736 | Focado como modelo final pela eficiência e consistência sólida. |
| LightGBM Classifier | 0.5722 | Altamente eficiente e performático. |
| Decision Tree Classifier | 0.5714 | Modelo simples, porém mais sensível a overfitting. |
| CatBoost Classifier | 0.5509 | Lida bem com variáveis categóricas sem pré-processamento complexo. |
| Logistic Regression | 0.5234 | Modelo linear clássico, útil pela interpretabilidade. |
| *Baseline (DummyClassifier)* | *0.4704* | *Piso de desempenho; prevê sempre a classe majoritária.* |

> 📌 **Conclusão de Performance:** Todos os modelos candidatos superaram o baseline matemático (`0.4704`) por uma margem estatística significativa, provando que as features selecionadas carregam sinal preditivo real.

### 💡 Validação de Hipóteses:
* A extrema importância das colunas `descricaoMonocratica` e `texto_integral` foi confirmada, justificando a blindagem contra o *data leakage*.
* A relevância do **Nome do Ministro Relator** (`NM_MINISTRO`) e dos termos de assunto foi validada pela EDA e pelo ganho de importância de atributos (*Feature Importance*).

---

## 🛑 Limitações do MVP

* **Janela de Texto Reduzida:** A alta taxa de ausência em `texto_integral` impossibilitou o uso de técnicas de PLN mais robustas (como Transformers/BERT) nas decisões completas.
* **Viés Histórico:** O modelo tende a replicar os vieses jurisprudenciais inerentes ao histórico de decisões dos ministros.
* **Janela Temporal:** Mudanças bruscas na legislação ou alteração de entendimento da Corte podem degradar a performance do modelo a médio prazo.

---

## 🔮 Próximos Passos

- [ ] **Otimização Avançada:** Implementar busca exaustiva de hiperparâmetros utilizando Otimização Bayesiana através da biblioteca **Optuna**.
- [ ] **Técnicas de Reamostragem:** Testar abordagens de oversampling/undersampling como `SMOTE` ou `ADASYN` no pipeline.
- [ ] **Análise Qualitativa de Erros:** Avaliar os resíduos e predições incorretas para mapear novos padrões jurídicos e gerar novas features (*feature engineering*).
- [ ] **Pipeline de MLOps:** Implementar monitoramento contínuo para re-treinamento do modelo conforme novas decisões do STJ forem publicadas.

---

## 🛠️ Tecnologias e Bibliotecas

* **Linguagem:** Python 3.x
* **Manipulação e Análise:** Pandas, NumPy
* **Machine Learning:** Scikit-learn, XGBoost, LightGBM, CatBoost
* **Processamento de Texto (NLP):** NLTK
* **Visualização:** Matplotlib, Seaborn
* **Infraestrutura:** Google Colab Environment

---

## 📂 Como Executar o Projeto

1.  **Clone o Repositório:**
    ```bash
    git clone [https://github.com/rafaeldebarros/MVP_machine_learning.git](https://github.com/rafaeldebarros/MVP_machine_learning.git)
    ```
2.  **Ambiente de Execução:** O arquivo principal `MVP_ML_DL.ipynb` foi projetado para rodar diretamente no **Google Colab**.
3.  **Execução:** Abra o arquivo `.ipynb` no Colab e execute as células sequencialmente. O script fará o download automatizado das dependências e bases de dados necessárias.

---

## 👤 Autor

* **Rafael Souza de Barros**
* **Data de Atualização:** 01/07/2026
