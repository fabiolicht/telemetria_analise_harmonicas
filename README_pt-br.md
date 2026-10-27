# CCeAE 2026 — Análise de Qualidade de Energia e Telemetria IoT
Repositório de scripts Python para **caracterização estatística de datasets elétricos**, **simulação de sensores IoT via MQTT** e **suporte analítico** ao artigo acadêmico do projeto CCeAE 2026 (Computação Científica Aplicada à Engenharia Elétrica).
O projeto aborda monitoramento de qualidade de energia em sistemas trifásicos, com foco em distorção harmônica (THD), análise espectral (FFT) e telemetria em tempo real na borda.
---
## Sumário
- [Visão geral](#visão-geral)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Análise dos códigos](#análise-dos-códigos)
- [Datasets](#datasets)
- [Requisitos e instalação](#requisitos-e-instalação)
- [Como executar](#como-executar)
- [Saídas geradas](#saídas-geradas)
- [Fundamentação matemática](#fundamentação-matemática)
- [Resultados esperados](#resultados-esperados)
- [Limitações e melhorias sugeridas](#limitações-e-melhorias-sugeridas)
- [Referências](#referências)
- [Licença e autoria](#licença-e-autoria)
---
## Visão geral
O repositório contém três módulos Python complementares:
| Script | Propósito | Dataset |
|--------|-----------|---------|
| `Eletrica.py` | Análise exploratória de metadados de sinais harmônicos sintéticos | `harmonic_signals_dataset_metadata.csv` |
| `Eletrica_V2.py` | Caracterização de medições reais de um data center universitário (UCO) | `Dataset_data_center.xlsx` |
| `EletricaSensorVirtual.py` | Simulador de sensor ESP32 publicando tensão/corrente via MQTT | Geração sintética em tempo real |
Existem duas versões do artigo LaTeX empacotadas nos arquivos ZIP:
- **`CCeAE_2026_Eletrica.zip`** — foco no dataset de sinais harmônicos injetados
- **`CCeAE_2026_Eletrica_V2.zip`** — foco no dataset real do data center da Universidade de Córdoba
---
## Estrutura do repositório
```
Eletrica/
├── Eletrica.py                              # Análise V1 — dataset harmônico (CSV)
├── Eletrica_V2.py                           # Análise V2 — data center (XLSX)
├── EletricaSensorVirtual.py                 # Simulador MQTT de sensor elétrico
├── harmonic_signals_dataset_metadata.csv    # Metadados de 2.012 campanhas de injeção
├── Dataset_data_center.xlsx                 # 11.088 amostras reais (77 dias)
├── Caracterizacao_Dataset_Harmonicas.png    # Figura gerada por Eletrica.py
├── dataset_caracterizacao_uco.png           # Figura gerada por Eletrica_V2.py
├── CCeAE_2026_Eletrica.pdf                  # Artigo compilado (V1)
├── CCeAE_2026_Eletrica.zip                  # Fonte LaTeX + figura V1
├── CCeAE_2026_Eletrica_V2.zip               # Fonte LaTeX + figuras V2
└── README.md
```
---
## Análise dos códigos
### `Eletrica.py` — Caracterização do dataset harmônico
**Função principal:** `analisar_dataset_metadados(caminho_csv)`
Fluxo de execução:
1. Valida a existência do arquivo CSV de metadados.
2. Carrega o dataset com `pandas.read_csv`.
3. Imprime resumo estatístico: total de registros, colunas e estatísticas descritivas de `tone_frequency_hz`.
4. Gera uma figura com dois painéis:
   - **Painel A:** histograma com KDE das frequências de injeção (`tone_frequency_hz`).
   - **Painel B:** gráfico de barras horizontais (`countplot`) das categorias de forma de onda (`category1`).
5. Salva a figura em `Caracterizacao_Dataset_Harmonicas.png` (300 DPI) e exibe na tela.
**Pontos fortes:**
- Código modular com função reutilizável e bloco `if __name__ == "__main__"`.
- Verificação de existência do arquivo antes do processamento.
- Verificação defensiva de colunas (`if 'tone_frequency_hz' in df.columns`).
- Configuração global de estilo matplotlib para publicação acadêmica.
**Observações:**
- O rótulo do eixo Y no painel A diz "Frequência Absoluta (Amostras)", o que está correto para um histograma.
- O script analisa apenas **metadados**; os sinais brutos (`.h5`, `.bin`) referenciados no CSV não estão incluídos neste repositório.
---
### `Eletrica_V2.py` — Caracterização do data center (UCO)
Script linear (sem encapsulamento em função) dedicado ao dataset real da Universidade de Córdoba.
Fluxo de execução:
1. Configura tema seaborn (`whitegrid`) e cria figura 1×2.
2. Lê `Dataset_data_center.xlsx` com `pandas.read_excel`.
3. **Painel A:** histograma + KDE de `ptotal` (potência ativa total em W).
4. **Painel B:** boxplot das correntes RMS trifásicas (`irmsa`, `irmsb`, `irmsc`), com `melt` para formato longo.
5. Salva `dataset_caracterizacao_uco.png` (300 DPI).
6. Imprime estatísticas descritivas de `ptotal`, `irmsa`, `irmsb`, `irmsc`, `thdva`, `thdia`.
**Pontos fortes:**
- Visualização clara do desequilíbrio entre fases via boxplot.
- Uso de LaTeX nos títulos (`$P_{total}$`) para formatação acadêmica.
- Paleta de cores consistente por fase (azul, verde, âmbar).
**Observações:**
- Caminhos de arquivo são hardcoded; executar sempre a partir do diretório raiz do projeto.
- O arquivo XLSX tem ~26 MB; a leitura pode demorar em máquinas mais lentas.
- Colunas `thdva` e `thdia` correspondem a THD de tensão e corrente, respectivamente, já presentes no dataset (não calculadas neste script).
---
### `EletricaSensorVirtual.py` — Simulador de sensor IoT (MQTT)
Simula um sensor ESP32 que publica leituras elétricas em tempo real no broker público Mosquitto.
**Parâmetros configuráveis:**
| Parâmetro | Valor | Descrição |
|-----------|-------|-----------|
| `BROKER_ADDRESS` | `test.mosquitto.org` | Broker MQTT público |
| `PORT` | `1883` | Porta MQTT padrão |
| `TOPIC` | `telemetria/motor_01/eletrica` | Tópico de publicação |
| `FREQ_REDE` | `60.0 Hz` | Frequência fundamental |
| `TAXA_AMOSTRAGEM` | `1000.0 Hz` | Taxa de amostragem simulada |
**Função `gerar_amostra(t)`:**
- Gera tensão fundamental: 220 V RMS + ruído gaussiano (σ = 2 V).
- Gera corrente fundamental: 15 A RMS + ruído gaussiano (σ = 0,5 A).
- **Injeção de falha:** quando `int(t) % 10 >= 5`, adiciona componente do **5º harmônico** (300 Hz) com amplitude de 40 V na tensão — simula distorção harmônica periódica.
**Payload JSON publicado:**
```json
{"tensao_v": 311.42, "corrente_a": 21.15}
```
**Pontos fortes:**
- Usa API MQTT v2 (`CallbackAPIVersion.VERSION2`).
- Tratamento de `KeyboardInterrupt` para encerramento limpo.
- `time.sleep(0.001)` limita a taxa efetiva de publicação (~1 kHz teórico, ~1000 msg/s na prática).
**Observações:**
- O comentário no código menciona "a cada 5 segundos", mas a lógica `% 10 >= 5` alterna falha nos **últimos 5 segundos de cada ciclo de 10 s** (50% do tempo).
- Depende de conexão de rede com o broker público; não adequado para produção.
- Não há autenticação TLS — apenas para testes/demonstração.
---
## Datasets
### 1. `harmonic_signals_dataset_metadata.csv`
Metadados de campanhas de injeção de sinais harmônicos em sistemas elétricos.
| Campo | Descrição |
|-------|-----------|
| `timestamp` | Data/hora da coleta |
| `collected_signal_filename` | Arquivo `.h5` do sinal coletado |
| `injected_signal_filename` | Arquivo `.bin` do sinal injetado |
| `label` | Identificador da campanha |
| `tone_frequency_hz` | Frequência do tom de identificação (Hz) |
| `category1` | Tipo de forma de onda injetada |
| `frequency1_hz` | Faixa ou valor de frequência do sinal |
| `category2`, `frequency2_hz` | Categoria/frequência secundária (quando aplicável) |
| `description` | Descrição textual |
**Estatísticas:**
| Métrica | Valor |
|---------|-------|
| Total de registros | 2.012 |
| Categorias principais | Square (643), Triangle (510), Sine (419), Square_75_duty (348), Chirp (92) |
| Faixa de frequências | ~4 kHz – ~14 kHz (multimodal) |
