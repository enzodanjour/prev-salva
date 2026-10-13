# Dicionário de Dados — Manutenção Preditiva (Indústria 4.0)

Este documento descreve cada coluna do conjunto de dados `data/raw/manutencao_preditiva.csv`, sua unidade de medida, tipo primitivo e papel na modelagem preditiva.

| Coluna | Tipo | Unidade | Descrição | Papel na Modelagem |
| :--- | :--- | :--- | :--- | :--- |
| `udi` | Inteiro | N/A | Identificador único sequencial do registro (1 a 10000). | **Descarte**: Não possui poder preditivo e gera memorização. |
| `id_produto` | String | N/A | Código do produto composto pelo tipo e número de série (ex: `M14860`). | **Descarte**: Identificador de alta cardinalidade sem poder de generalização. |
| `tipo` | String (Categórica) | N/A | Variante de qualidade do equipamento (`L` = Low/60%, `M` = Medium/30%, `H` = High/10%). | **Preditora**: Mapeada ordinalmente (`L: 0, M: 1, H: 2`). |
| `temperatura_ar_k` | Float | Kelvin (K) | Temperatura ambiente ao redor do equipamento monitorado. | **Preditora**: Contínua. |
| `temperatura_processo_k` | Float | Kelvin (K) | Temperatura de operação interna do maquinário gerada pelo processo produtivo. | **Preditora**: Contínua. |
| `velocidade_rotacao_rpm` | Float | Rotações por Minuto (RPM) | Velocidade de rotação calculada a partir de uma potência gerada de 2860 W. | **Preditora**: Contínua. Apresenta assimetria positiva acentuada. |
| `torque_nm` | Float | Newton-metro (N·m) | Torque resistente exercido no maquinário com distribuição simétrica. | **Preditora**: Contínua. |
| `desgaste_ferramenta_min` | Inteiro | Minutos (min) | Tempo de uso acumulado da ferramenta de corte durante o ciclo de usinagem. | **Preditora**: Contínua. |
| `falha_maquina` | Inteiro (Binário) | Booleano (0 ou 1) | Indicador geral de falha do maquinário. `1` = Quebra/Avaria; `0` = Operação Normal. | **Variável Alvo (`y`)**: Desbalanceada (~3,39% classe positiva). |
| `falha_twf` | Inteiro | Booleano (0 ou 1) | Falha por desgaste da ferramenta (*Tool Wear Failure*). | **Descarte (Data Leakage)**: Modo específico de falha posterior ao evento. |
| `falha_hdf` | Inteiro | Booleano (0 ou 1) | Falha por dissipação de calor (*Heat Dissipation Failure*). | **Descarte (Data Leakage)**: Modo específico de falha posterior ao evento. |
| `falha_pwf` | Inteiro | Booleano (0 ou 1) | Falha de potência (*Power Failure*). | **Descarte (Data Leakage)**: Modo específico de falha posterior ao evento. |
| `falha_osf` | Inteiro | Booleano (0 ou 1) | Falha por sobrecarga (*Overstrain Failure*). | **Descarte (Data Leakage)**: Modo específico de falha posterior ao evento. |
| `falha_rnf` | Inteiro | Booleano (0 ou 1) | Falha aleatória (*Random Failure*). | **Descarte (Data Leakage)**: Modo específico de falha posterior ao evento. |
| `potencia_w` | Float | Watts (W) | Potência mecânica derivada calculada como `torque_nm * velocidade_rotacao_rpm * (2 * pi / 60)`. | **Feature Derivada**: Engenharia de atributos (Fase 3). |
