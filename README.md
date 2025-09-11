# **Documentação Técnica do Dashboard**

## Gestão de Manutenção

**Data de início do projeto:** 17 de ago. de 2025

**Data da última atualização da documentação:** 08 de set. de 2025

**Desenvolvido por:** Rosberg Alves

## 01 Visão Geral

### 1.1 Objetivo do Dashboard

Criar um dashboard analítico em Power BI que permita acompanhar e avaliar o desempenho da manutenção da fábrica, com foco em:

* Monitorar tempo total das manutenções e tempo de execução real.
* Identificar gargalos em máquinas/equipamentos.
* Avaliar custos de peças por manutenção.
* Classificar manutenções por rank e consequência.
* Permitir visão por centro de custo, tipo de equipamento e período.

### 1.2 Público-alvo

* **Primário:** Coordenação de Manutenção
* **Secundário:** Gerencia Industrial
* **Usuários Finais:** Equipe de Manutenção

### 1.3 Contexto do Negócio

Um fabricante multinacional de pneus, que contém informações de janeiro de 2015 até agosto de 2022.

### 1.4 Frequência de Uso Esperada

* **Diária:** Monitoramento dos principais indicadores.

## 02 Fonte de Dados

### 2.1 Infraestrutura Técnica

* **Arquivo CSV:** Excel
* **ETL:** Power Query
* **Ferramenta de Visualização:** Power BI

## 03 Transformação dos Dados

### 3.1 Processo de ETL

O processo de transformação utiliza Power Query estruturando os dados em três etapas principais.

### Camada Bronze

* **Objetivo:** Armazenar os dados brutos, preservando a integridade do arquivo original sem aplicar transformações, mantendo assim a fidelidade da fonte.

* **Fonte dos Dados:**
    * **Arquivo:** “Manutenção_Industria_Pneus.csv”
    * **Formato:** CSV (valores separados por vírgula).
    * **Origem:** Dataset público do Kaggle (ordens de manutenção de uma fabricante multinacional de pneus).
    * **Período:** Janeiro/2015 até Agosto/2022.

* **Método de Ingestão**
    * **Ferramenta:** Power BI Desktop
    * **Conector Utilizado:** Texto/CSV
    * **Justificativa da Escolha:** O dataset é único e estático, não haverá substituições nem cargas incrementais.

* **Passos de Ingestão**
    * No Power BI Desktop, acessar Página Inicial → Obter Dados → Texto/CSV.
    * Selecionar o arquivo “Manutenção_Industria_Pneus.csv” no diretório local.
    * Confirmar a importação mantendo o encoding padrão (UTF-8) e delimitador vírgula ( , ).
    * Nomear a consulta como “Camada Bronze” que será a base para criação da camada prata.

### Camada Prata

* **Objetivo:** A Camada Prata contém os dados tratados e padronizados da tabela de ordens de manutenção, mantendo a rastreabilidade das informações originais, incluindo colunas auxiliares e flags de qualidade. Essa camada serve como base confiável para a criação da Camada Ouro (modelo estrela) no Power BI.

* **Processos:**
    * Corrigir tipos de dados (datas, horas, valores).
    * Criar colunas auxiliares.
    * Padronizar texto (equipamento, centro de custo).
    * Remover inconsistências (linhas vazias, duplicadas).
    * Nomear como Consulta Prata.

### Tratamentos Aplicados

* **Padronização de Colunas de Texto**
    * Todas os registros de texto das tabelas Dimensão foram convertidos para Primeira Linha Maiúsculo.
    * Objetivo: Evitar inconsistências e variações de escrita.

* **Substituição de Valores Nulos**
    * Valores vazios na coluna NumeroOrdem foram substituídos por Sem_Ordem.
    * Valores vazios na coluna Rank, foram substituídos por Sem_Rank.

* **Criação de coluna derivada do Status da Consequencia**
    * Coluna StatusOcorrencia criada a partir da coluna Ocorrencia.
        * 1 = Máquina Parada
        * 2 = Máquina Parada Parcialmente
        * 3 = Máquina Funcionando
        * 4 = Não Informado
        * 5 = Não Informado
        * 6 = Não Informado
        * 7 = Não Informado
        * 8 = Não Informado
        * Objetivo: Facilitar a interpretação e análise operacional.

* **Criação de coluna derivada da Descrição do Rank**
    * Coluna DescricaoRank criada a partir da coluna Rank.
        * A = Manutenções com duração maiores ou iguais à 240 minutos
        * B = Manutenções com duração entre 20 e 240 minutos
        * C = Manutenções com duração entre 0 e 20 minutos
        * D = Manutenção de máquinas específicas independente do tempo
        * Sem_Rank = Sem classificação

* **Criação de colunas auxiliares de data e hora**
    * DataHoraInicio = DataInicio + HoraInicio
    * DataHoraOrdem = DataInicio + HoraOrdem
    * DataHoraFim = DataFim + HoraFim
    * Tratamento condicional: se hora for nula → valor null.
    * Objetivo: Possibilitar cálculo de métricas temporais como tempo de execução, tempo total.

* **Flags de qualidade**
    * SemOrdem = True se NumeroOrdem for nulo.
    * SemRank = True se Rank for nulo.
    * SemHoraInicio, SemHoraOrdem, SemHoraFim = True se respectivas colunas de hora forem nulas.
    * SemDataFim = True se DataFim for nula.
    * Objetivo: Analisar ordens concluídas e ordens abertas.

* **Tratamento de métricas numéricas**
    * Coluna CustoPecas verificada, sem valores negativos.
    * Colunas TempoReal e TempoManutencao corrigidas para número decimal, separador correto ( , ).
    * Objetivo: Garantir consistência para cálculos e visualizações.

* **Verificação de duplicatas**
    * NumeroNota e NumeroOrdem conferidos e não apresentaram duplicatas.
    * Mantidos como chaves naturais de referência.

* **Linhas com dados incompletos**
    * Mantidas na consulta para permitir análise de ordens abertas e backlog operacional.

### Observações Gerais

* A Camada Prata não possui ID’s nem dimensões, sua função é fornecer dados limpos e confiáveis para modelagem analítica na Camada Ouro.
* Mantém a rastreabilidade dos dados originais e registros incompletos, permitindo análises operacionais e históricas.
* Facilita criação de tabelas fato e dimensões, incluindo calendário, equipamentos, centros de custo, ranks e status de consequência.

### Camada Ouro

* **Estrutura do Modelo**
    * **Tabela Fato (FtManutencao):** Centraliza os registros das ordens de manutenção, contendo métricas quantitativas (ex.: TempoReal, CustoPecas) e as chaves de ligação com as dimensões.

    * **Tabelas Dimensão:** Fornecem contexto descritivo para análise como tempo, equipamentos, centro de custo, ocorrência, rank, hora, documento etc.

* **Relacionamentos**
Cada Dimensão possui um relacionamento 1:N com a tabela fato FtManutencao, a direção do filtro é do tipo single da dimensão para a fato, preservando a integridade do modelo estrela sendo:
    * **DimCalendario:** Permite análise temporal em diferentes granularidades (ano, mês, dia).
    * **DimEquipamento:** Identifica de forma única os equipamentos e máquinas da fábrica.
    * **DimCentroCusto:** Possibilita segmentação financeira e administrativa.
    * **DimOcorrencia:** Categoriza o status das ocorrências (parada, funcionando, não informado).
    * **DimRank:** Classifica as manutenções em grupos (A, B, C, D, Sem Rank).
    * **DimHora:** Auxilia análises de ocorrências por horário e turno.
    * **DimDocumento:** Garante integridade e rastreabilidade de ordens e notas.

* **Processo:** Tabelas de Dimensão criadas.

    * **DimCalendario:** Dimensão temporal que fornece granularidade de análise ao modelo, permitindo relacionar os eventos da Fato com informações de tempo em diferentes níveis (ano, trimestre, mês, semana, dia etc.). Foi criada de forma dinâmica, baseada nos limites de datas existentes na Camada Prata (colunas DataInicio e DataFim), garantindo escalabilidade e atualização automática do período de análise.
    * **Origem:** Consulta nula criada no Power Query.
    * **Lógica aplicada:** 
        * DataMinima = menor valor encontrado em DataInicio (camada Prata).
        * DataMaxima = maior valor encontrado em DataFim (Camada Prata).
        * DataInicial = 1º de janeiro do ano correspondente à data mínima.
        * DataFim = 31 de dezembro do ano correspondente à data máxima.
        * Duracao = intervalo de dias entre DataInicial e DataFinal.
        * List.Dates utilizado para gerar a sequência de datas contínuas.
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Calendario, ID_DataInicio e ID_DataFim.
    * **Colunas Principais:**
        * Data: Data no formato completo (DD-MM-YYYY).
        * Ano: Ano da data.
        * Mes: Número do mês.
        * NomeMes: Nome do mês por extenso.
        * Trimestre: Identificação do trimestre (1, 2, 3, 4).
        * DiaDaSemana: Nome do dia da semana.
        * DiaUtil: Flag indicando se o dia é útil ("Sim" ou "Não").

    * **DimEquipamento:** Dimensão de apoio ao modelo, responsável por armazenar a lista de equipamentos existentes nas ordens de manutenção. Permite análises relacionadas ao desempenho, falhas e impactos específicos por equipamento.
    * **Origem:** Camada Prata (coluna Equipamento).
    * **Lógica aplicada:**
        * Seleção distinta dos valores únicos.
        * Padronização textual (primeira letra maiúscula, remoção de espaços).
        * Inclusão de chave substituta (ID_Equipamento).
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Equipamento.
    * **Colunas Principais:**
        * Equipamento: Indica o máquina que necessita de manutenção.

    * **DimCentroCusto:** Dimensão responsável por armazenar a lista única de centros de custo da fábrica. Permite análises por setor ou área de operação, e facilita a segmentação das métricas de manutenção por localização administrativa.
    * **Origem:** Camada Prata (coluna CentroCusto).
    * **Lógica aplicada:**
        * Seleção distinta dos valores únicos.
        * Padronização textual (primeira letra maiúscula, remoção de espaços).
        * Inclusão de chave substituta (ID_CentroCusto).
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_CentroCusto.
    * **Colunas Principais:**
        * CentroCusto: Centro de custo de onde a solicitação de manutenção está sendo emitida.

    * **DimOcorrencia:** Armazena os diferentes tipos de ocorrências associadas às ordens de manutenção. Essa dimensão é fundamental para categorizar os impactos e facilitar a análise de criticidade das ocorrências registradas.
    * **Origem:** Camada Prata (coluna Consequencia).
    * **Lógica aplicada:**
        * Seleção de valores distintos.
        * Padronização textual (primeira letra maiúscula, remoção de espaços).
        * Inclusão de chave substituta (ID_Ocorrencia).
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Ocorrencia.
    * **Colunas Principais:**
        * Ocorrencia: Ao gerar uma nota, o colaborador tem que colocar uma ocorrência.
        * StatusOcorrencia: Descrição da situação do equipamento no ato da ocorrência.

    * **DimRank:** Armazena a categorização de manutenções realizadas manualmente pela equipe de manutenção, com base no tempo de execução ou em características específicas das máquinas.
    * **Origem:** Camada Prata (coluna Rank).
    * **Lógica aplicada:**
        * Seleção de valores distintos.
        * Padronização textual (primeira letra maiúscula, remoção de espaços).
        * Inclusão de chave substituta (ID_Rank).
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Rank.
    * **Colunas Principais:**
        * Rank: Input manual que a equipe de manutenção faz para categorizar o tempo necessário para e execução.
        * DescricaoRank: Critério de classificação dos tipos de ocorrencias dividido em A, B, C e D.

    * **DimHora:** Fornece a dimensão de tempo por hora do dia, permitindo a análise das manutenções por hora exata, período do dia (manhã, tarde, noite, madrugada). É usada para vincular os horários de início, ordem e fim das manutenções à dimensão de tempo.
    * **Origem:** Criada via Consulta Nula no Power Query (tabela sintética).
    * **Lógica aplicada:** Valores inteiros de 0 a 23, convertidos para hora do dia e categorizados por turno e inclusão da linha especial 25 = Sem_Hora para permitir relacionamento mesmo em casos de valores nulos na fato.
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Hora, ID_HoraInicio, ID_HoraOrdem e ID_HoraFim.
    * **Colunas Principais:**
        * Hora: Horas do dia de 00:00 a 23:00.
        * HoraDoDia: Definição de manhã, tarde, noite e madrugada com base na hora.
        * Turno: Definição do turno baseado na hora.

    * **DimDocumento:** Contém informações únicas sobre cada documento ou ordem de manutenção, permitindo relacionar a fato FtManutencao de forma otimizada.
    * **Origem:** Camada Prata (coluna NumeroNota, NumeroOrdem).
    * **Relacionamento:** Conectada à tabela FtManutencao via chave ID_Documento.
    * **Colunas Principais:**
        * NumeroNota: Número da nota gerada ao solicitar uma manutenção.
        * NumeroOrdem: Número de ordem gerada pela equipe de manutenção, ao atender uma solicitação.

* **Processo:** Tabela de Fatos criada.

    * **FtManutencao:** Consolida os registros de ordens de manutenção, com granularidade 1 linha por manutenção. Contém as chaves de relacionamento com as dimensões Calendario, Equipamento, CentroCusto, Ocorrencia, Rank, Hora e Documento.
    * **Origem:** Consulta Prata (Manutenção_Industria_Pneus_Prata), proveniente da base operacional da manutenção.
    * **Colunas Principais:**
        * ID_Manutencao: Identificador único da manutenção na tabela fato.
        * ID_Calendario: Chave estrangeira para DimCalendario, representando a data de abertura/ocorrência.
        * ID_Equipamento: Chave estrangeira para DimEquipamento.
        * ID_CentroCusto: Chave estrangeira para DimCentroCusto.
        * ID_Ocorrencia: Chave estrangeira para DimOcorrencia, que classifica o status da manutenção (parada, funcionando, parcial).
        * ID_Rank: Chave estrangeira para DimRank, utilizada para análises de priorização ou classificação.
        * ID_Hora: Chave estrangeira para DimHora, detalhando o horário de início/fim da manutenção.
        * ID_Documento: Chave estrangeira para DimDocumento, vinculada ao documento de origem ou ordem de serviço.
    * **Colunas de tempo e desempenho:**
        * ID_HoraInicio: Horário em que a manutenção começou (extraído do sistema de ordens).
        * ID_HoraOrdem: Horário em que a ordem foi registrada/aberta.
        * ID_HoraFim: Horário de término da manutenção.
        * ID_DataInicio: A data em que foi aberta a solicitação de manutenção.
        * ID_DataFim: Data do fim da manutenção.
        * TempoReal: Tempo total de manutenção (em minutos), considerado para indicadores de parada, funcionamento e disponibilidade.
        * TempoManutencao: Tempo líquido da atividade de manutenção (em minutos), desconsiderando intervalos ou atrasos de registro.
        * TempoReal: O tempo que levou desde a solicitação até o fim da manutenção.
        * CustoPecas: Valor total de peças utilizadas na manutenção.

### Observações Gerais

* As colunas de tempo (TempoReal, TempoManutencao) foram padronizadas para minutos, mas todas as medidas DAX convertem para horas quando necessário.
* O campo ID_Ocorrencia é central para os cálculos de disponibilidade, pois diferencia se a manutenção implicou em parada total, funcionamento normal ou parada parcial.

### 3.2 Modelo Dimensional

**Modelo Conceitual:** Criado no software brModelo antes do desenvolvimento no Power BI, serviu como guia para definição das entidades, atributos e relacionamentos.

![Modelo Conceitual](https://raw.githubusercontent.com/rosbergalves/gestaomanutencao_dashboard/refs/heads/main/diagramas/Diagrama_Conceitual.png)

**Modelo Estrela (Star Schema):** O modelo foi desenvolvido no formato estrela (Star Schema) para facilitar consultas analíticas, manter simplicidade de relacionamento, tipos de cardinalidade e ajustes de integridade referencial garantindo boa performance em cálculos DAX.

![Modelo Estrela (Star Schema)](https://raw.githubusercontent.com/rosbergalves/gestaomanutencao_dashboard/refs/heads/main/diagramas/Modelo_Dimensional.png)

### 3.3 Medidas DAX

### Disponibilidade
```DAX
Disponibilidade =
DIVIDE([Horas_Funcionando], [Horas_Disponiveis], 0)

Descrição: Calcula a disponibilidade operacional dos equipamentos, dividindo as horas em funcionamento pelas horas disponíveis.
```

### Horas_Disponiveis
```DAX
Horas_Disponiveis =
VAR TurnosPorDia = 3  -- total de turnos por dia
VAR HorasPorTurno = 7 -- horas planejadas por turno
VAR QtdEquipamentos =
    DISTINCTCOUNT(DimEquipamento[ID_Equipamento])
VAR DiasUteis =
    CALCULATE(
        COUNTROWS(DimCalendario),
        KEEPFILTERS(DimCalendario[DiaUtil] = "Sim")
    )
RETURN
    QtdEquipamentos * DiasUteis * TurnosPorDia * HorasPorTurno

Descrição: Horas totais disponíveis considerando dias úteis, quantidade de equipamentos, turnos e horas por turno.
```

### Horas_Funcionando
```DAX
Horas_Funcionando =
CALCULATE(
    SUM(FtManutencao[TempoReal]),
    FILTER(
        DimOcorrencia,
        DimOcorrencia[Ocorrencia] = 3
    )
)

Descrição: Soma do tempo real de funcionamento, considerando ocorrências de tipo 3 (funcionamento normal).
```

### Horas_Paradas
```DAX
Horas_Paradas =
CALCULATE(
    SUM(FtManutencao[TempoReal]),
    KEEPFILTERS(
        FILTER(DimOcorrencia, DimOcorrencia[Ocorrencia] IN {1})
    )
)

Descrição: Soma do tempo de paradas, considerando ocorrências de tipo 1.
```

### Horas_Operacionais
```DAX
Horas_Operacionais =
VAR hDisp = [Horas_Disponiveis]
VAR hPar  = [Horas_Paradas]
RETURN
    MAX(0, hDisp - hPar)

Descrição: Horas disponíveis menos horas paradas, garantindo que não fique negativo.
```

### Custo_CentroCusto
```DAX
Custo_CentroCusto =
SUM(FtManutencao[CustoPecas])

Descrição: Soma dos custos de peças, analisado por centro de custo.
```

### Custo_Equipamento
```DAX
Custo_Equipamento =
SUM(FtManutencao[CustoPecas])

Descrição: Soma dos custos de peças, analisado por equipamento.
```

### CustoTotal_Manutencao
```DAX
CustoTotal_Manutencao =
SUM(FtManutencao[CustoPecas])

Descrição: Soma total histórica dos custos de peças em manutenções.
```

### MTBF (Mean Time Between Failures)
```DAX
MTBF =
VAR falhas = [Qtd_Falhas]
VAR horasOp = [Horas_Operacionais]
RETURN
    IF(falhas = 0, BLANK(), DIVIDE(horasOp, falhas))

Descrição: Tempo médio entre falhas: divide horas operacionais pela quantidade de falhas.
```

### MTTR (Mean Time To Repair)
```DAX
MTTR =
VAR falhas = [Qtd_Falhas]
VAR sumRep =
    CALCULATE(
        SUM(FtManutencao[TempoManutencao]),
        KEEPFILTERS(
            FILTER(DimOcorrencia, DimOcorrencia[Ocorrencia] IN {1, 2})
        )
    )
RETURN
    IF(falhas = 0, BLANK(), DIVIDE(sumRep, falhas))

Descrição: Tempo médio para reparo: divide o tempo total de manutenção pela quantidade de falhas.
```

### Qtd_Falhas
```DAX
Qtd_Falhas =
CALCULATE(
    COUNTROWS(FtManutencao),
    KEEPFILTERS(
        FILTER(DimOcorrencia, DimOcorrencia[Ocorrencia] IN {1, 2})
    )
)

Descrição: Número total de falhas (ocorrências do tipo 1 e 2).
```

### Qtd_EquipamentosFalha
```DAX
Qtd_EquipamentosFalha =
CALCULATE(
    DISTINCTCOUNT(FtManutencao[ID_Equipamento]),
    FILTER(
        DimOcorrencia,
        DimOcorrencia[Ocorrencia] = 2
    )
)

Descrição: Número de equipamentos distintos que apresentaram falha (Ocorrencia = 2).
```

### Qtd_Ocorrencias
```DAX
Qtd_Ocorrencias =
COUNTROWS(FtManutencao)

Descrição: Contagem de todas as ocorrências registradas.
```

### Qtd_OcorrenciasEmAberto
```DAX
Qtd_OcorrenciasEmAberto =
VAR vDataFinal = MAX(DimCalendario[Data])
VAR vOcorrenciasAbertas =
    FILTER(
        ALL(FtManutencao),
        FtManutencao[ID_DataInicio] <= vDataFinal &&
        (FtManutencao[ID_DataFim] > vDataFinal || ISBLANK(FtManutencao[ID_DataFim]))
    )
RETURN
    COUNTROWS(vOcorrenciasAbertas) + 0

Descrição: Quantidade de ocorrências que não possuem data de fim ou que ultrapassam a data de análise.
```

### TempoMedio_Manutencao
```DAX
TempoMedio_Manutencao =
AVERAGE(FtManutencao[TempoReal])
Descrição: Tempo médio das manutenções com base na coluna TempoReal na tabela FtManutencao.
```

### 3.4 Design Dashboard

O design do dashboard (capa, cartões personalizados e layout) foi desenvolvido previamente no Figma e replicado no Power BI.

**Capa**

![Capa](https://raw.githubusercontent.com/rosbergalves/gestaomanutencao_dashboard/refs/heads/main/design_dashboard_figma/Capa_Manutencao.png)

**Layout**

![Layout](https://raw.githubusercontent.com/rosbergalves/gestaomanutencao_dashboard/refs/heads/main/design_dashboard_figma/Pagina1_Manutencao.png)

**Cartões**

![Cartões](https://raw.githubusercontent.com/rosbergalves/gestaomanutencao_dashboard/refs/heads/main/design_dashboard_figma/Pagina1_Manutencao.png)

### 3.5 Links

[Dashboard em Produção](https://app.powerbi.com/view?r=eyJrIjoiZDFkZTAxNWItOGQ2Yy00MDgwLWJkMmEtZTZjYTc1ZmFiNmI3IiwidCI6IjIxYjFmMTkyLTUyM2MtNGQ5Ny1iYzc2LWYzZDgwYTYxYjNmNCJ9)
