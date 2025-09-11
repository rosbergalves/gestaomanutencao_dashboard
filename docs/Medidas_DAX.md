# Medidas DAX - Dashboard de Manutenção

Este documento lista todas as medidas criadas no modelo, com descrição e código DAX.

## Medidas DAX

### **Disponibilidade**
```DAX
Disponibilidade =
DIVIDE([Horas_Funcionando], [Horas_Disponiveis], 0)

Descrição: Calcula a disponibilidade operacional dos equipamentos, dividindo as horas em funcionamento pelas horas disponíveis.
```

### **Horas_Disponiveis**
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

### **Horas_Funcionando**
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

### **Horas_Paradas**
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

### **Horas_Operacionais**
```DAX
Horas_Operacionais =
VAR hDisp = [Horas_Disponiveis]
VAR hPar  = [Horas_Paradas]
RETURN
    MAX(0, hDisp - hPar)

Descrição: Horas disponíveis menos horas paradas, garantindo que não fique negativo.
```

### **Custo_CentroCusto**
```DAX
Custo_CentroCusto =
SUM(FtManutencao[CustoPecas])

Descrição: Soma dos custos de peças, analisado por centro de custo.
```

### **Custo_Equipamento**
```DAX
Custo_Equipamento =
SUM(FtManutencao[CustoPecas])

Descrição: Soma dos custos de peças, analisado por equipamento.
```

### **CustoTotal_Manutencao**
```DAX
CustoTotal_Manutencao =
SUM(FtManutencao[CustoPecas])

Descrição: Soma total histórica dos custos de peças em manutenções.
```
### **MTBF (Mean Time Between Failures)**
```DAX
MTBF =
VAR falhas = [Qtd_Falhas]
VAR horasOp = [Horas_Operacionais]
RETURN
    IF(falhas = 0, BLANK(), DIVIDE(horasOp, falhas))

Descrição: Tempo médio entre falhas: divide horas operacionais pela quantidade de falhas.
```

### **MTTR (Mean Time To Repair)**
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

### **Qtd_Falhas**
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

### **Qtd_EquipamentosFalha**
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

### **Qtd_Ocorrencias**
```DAX
Qtd_Ocorrencias =
COUNTROWS(FtManutencao)

Descrição: Contagem de todas as ocorrências registradas.
```

### **Qtd_OcorrenciasEmAberto**
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

### **TempoMedio_Manutencao**
```DAX
TempoMedio_Manutencao =
AVERAGE(FtManutencao[TempoReal])
Descrição: Tempo médio das manutenções com base na coluna TempoReal na tabela FtManutencao.
```




