# PrevFalha 4.0 — Manutenção Preditiva na Indústria 4.0

Pipeline preditivo ponta a ponta desenvolvido em Python para antecipação de falhas mecânicas em equipamentos industriais monitorados por sensores telemétricos.

---

## 1. Problema de Negócio
Em parques industriais modernos, a ocorrência de quebras mecânicas inesperadas acarreta elevados custos com interrupção de produção, manutenção emergencial e depreciação precoce do maquinário.
Este projeto implementa um modelo de Inteligência Artificial classificador para antecipar a ocorrência de falhas mecânicas (`falha_maquina = 1`) com base em telemetria em tempo real, permitindo manutenções preditivas controladas.

---

## 2. Dados e Granularidade
- **Fonte:** Conjunto histórico de monitoramento telemétrico (`data/raw/manutencao_preditiva.csv`), derivado do AI4I 2020 Predictive Maintenance Dataset (UCI Machine Learning Repository, licença CC BY 4.0).
- **Granularidade:** Cada linha representa uma leitura de ciclo produtivo em um equipamento fabril.
- **Dicionário de Atributos:** Documentação detalhada disponível em [`docs/dicionario_dados.md`](docs/dicionario_dados.md).

---

## 3. Tecnologias e Técnicas Utilizadas

### Stack Técnica
- **Linguagem:** Python 3.11+
- **Manipulação de Dados:** Pandas, NumPy
- **Visualização Analítica:** Matplotlib, Seaborn
- **Machine Learning & Pré-processamento:** Scikit-Learn (`train_test_split`, `StandardScaler`, `KNeighborsClassifier`, `DecisionTreeClassifier`, `DummyClassifier`, métricas)
- **Balanceamento de Dados:** Imbalanced-Learn (`SMOTENC`)
- **Ambiente:** Jupyter Notebook / Google Colab

### Arquitetura do Pipeline

```text
[data/raw/manutencao_preditiva.csv]
       │
       ▼
[Fase 1: EDA] ──► Dimensões, describe, histogramas, desbalanceamento (3.39%) e heatmap
       │
       ▼
[Fase 2: Data Prep] ──► Deduplicação, imputação estatística (média/mediana) e boxplots de outliers
       │
       ▼
[Fase 3: Feature Engineering] ──► Criação de 'potencia_w' (torque * rpm * 2pi / 60)
       │
       ▼
[Fase 4: Divisão & Balanceamento] ──► Descarte de data leakage, split estratificado 80/20 e SMOTENC no treino
       │
       ├─────────────────────────────────────────┐
       ▼ (Apenas Contínuas)                      ▼ (Escala Original)
[Fase 5: StandardScaler]                         [Árvore de Decisão]
(fit_transform treino / transform teste)                 │
       │                                                 │
       ▼                                                 ▼
[Fase 6: Tuning KNN]                             [Fase 6: Tuning Árvore]
(Variação de K ímpar: 1, 3, 5, 7, 9, 11, 15)     (Variação max_depth: 3, 5, 7, 10, None)
       │                                                 │
       └────────────────────┬────────────────────────────┘
                            ▼
[Fase 7: Avaliação e Veredito Final] ──► Acurácia no teste independente vs. Baseline Ingênuo
```

---

## 4. Estrutura do Repositório

```text
.
├── .gitignore                      # Regras de exclusão do Git (.venv, checkpoints)
├── requirements.txt                # Dependências fixadas do projeto
├── README.md                       # Documentação completa da solução
├── docs/
│   └── dicionario_dados.md         # Dicionário descritivo de colunas e unidades
├── data/
│   └── raw/
│       └── manutencao_preditiva.csv# Base de dados bruta (imutável)
└── notebooks/
    └── pipeline_manutencao_preditiva.ipynb # Notebook executável principal
```

---

## 5. Como Executar

### Opção A: No Google Colab (Recomendado na Nuvem)
1. Acesse o [Google Colab](https://colab.research.google.com/).
2. Clique na aba **Upload** e selecione o arquivo [`notebooks/pipeline_manutencao_preditiva.ipynb`](notebooks/pipeline_manutencao_preditiva.ipynb).
3. Caso a base não esteja carregada no ambiente temporário do Colab, o próprio notebook possui rotina automática na célula 0 para baixar o dataset diretamente do Google Drive oficial.
4. Clique em **Ambiente de execução > Executar tudo** (`Ctrl + F9`).

### Opção B: Localmente via Terminal
1. Clone o repositório:
   ```bash
   git clone https://github.com/SEU_USUARIO/prevfalha-4.0.git
   cd prevfalha-4.0
   ```
2. Crie e ative um ambiente virtual:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # No Windows: .venv\Scripts\activate
   ```
3. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```
4. Inicie o Jupyter e execute o notebook:
   ```bash
   jupyter notebook notebooks/pipeline_manutencao_preditiva.ipynb
   ```

---

## 6. Resultados e Veredito Técnico

| Modelo | Hiperparâmetro Ótimo | Acurácia no Teste | Comportamento de Generalização |
| :--- | :--- | :---: | :--- |
| **Baseline Ingênuo (Dummy)** | Classe majoritária (0) | 96,61% | Nulo (não detecta nenhuma falha real). |
| **KNN** | $K = 3$ | ~93,10% | Sensível à fronteira; sofre perda com dimensionalidade. |
| **Árvore de Decisão** | `max_depth = 5` | **~97,55%** | **Vencedor.** Supera o baseline e isola falhas sem sobreajuste. |

### Justificativa da Escolha da Árvore de Decisão:
1. **Acurácia Superior e Eficácia Real:** Supera o limiar de 96,61% da classe majoritária, minimizando falsos positivos e capturando com alta sensibilidade as quebras mecânicas.
2. **Explicabilidade:** Fornece regras transparentes de decisão ($x \le \theta$), auditáveis por engenheiros de manutenção na fábrica.
3. **Desempenho em Tempo Real:** Complexidade de inferência em $O(\text{profundidade})$, instantânea para sistemas SCADA/CLP industriais.

---

## 7. Melhorias Futuras
- **Pipelines do Scikit-Learn:** Encapsular todo o fluxo (`SimpleImputer`, `StandardScaler`, `SMOTE` e classificador) em um `imblearn.pipeline.Pipeline` integrado para prevenir vazamentos em validação cruzada.
- **Métricas Focadas na Falha:** Adoção de métricas de negócio como *Recall*, *F1-Score* ou *PR-AUC*, dado que o custo de uma falha não detectada (Falso Negativo) é drasticamente superior ao de um alarme falso.
- **Modelos de Ensemble:** Testar algoritmos baseados em comitês de árvores (*Random Forest*, *XGBoost* ou *LightGBM*) com calibração de probabilidades.

---

## 8. Link da Apresentação em Vídeo
- **Vídeo Explicativo do Projeto (Google Drive):** [Inserir link público do vídeo aqui]
