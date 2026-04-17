# Reconstrução de Base – People Analytics

Nesse projeto, o objetivo é **reconstruir a base de dados _People Analytics_** que contém informações fictícias de colaboradores mês a mês. A reconstrução se faz necessária porque a base original possui **inconsistências temporais**, como registros anteriores à admissão, posteriores ao desligamento e lacunas entre data de admissão e data de desligamento.

---

## Problema de negócio

Na análise inicial da base foram identificados casos como:

- Colaborador `ID 10219`  
  - Data Referência: 11/2021 a 12/2024  
  - Data Admissão: 12/2019  
  - Data Desligamento: Ativo até 12/2024  
  - De 12/2019 a 10/2021 não há registros.  

- Colaborador `ID 10052`  
  - Data Referência: 01/2021 a 12/2024  
  - Data Admissão: 03/2022  
  - Data Desligamento: Ativo até 12/2024  
  - De 01/2021 a 02/2022 há registros **extras**.  

- Colaborador `ID 10142`  
  - Data Referência: 07/2023 a 12/2024  
  - Data Admissão: 01/2019  
  - Data Desligamento: 04/2019  
  - De 07/2023 a 12/2024 há registros **extras** e de 01/2019 a 04/2019 há registros **faltantes**.

Essas inconsistências distorcem análises de headcount, desligamentos e salário por período, gerando resultados que não refletem a realidade do negócio. Apenas filtrar e remover linhas problemáticas poderia “maquiar” a base e comprometer decisões futuras.

---

## Objetivos do projeto

1. **Reconstruir a base de dados**, replicando as linhas uma quantidade de vezes igual à diferença de meses entre `Data_Admissao` e `Data_Desligamento` (ou data de corte), assumindo que essas datas estão corretas.  
2. **Gerar uma série mensal contínua por colaborador**, criando uma nova `Data_Referencia` para cada mês entre admissão e desligamento.  
3. **Resgatar as informações que variam no tempo** (como `Salario` e `Status_Emprego`) com base na **data de referência disponível mais próxima e anterior ou igual** à nova `Data_Referencia`.  
4. **Preencher as informações estáticas** (dimensões que não mudam no tempo, como `Nome_Funcionario`, `Departamento`, `Area`, `Cargo`, etc.) utilizando a última informação consistente por `ID_Funcionario`.

---

## Benefícios

- Base alinhada com a realidade temporal de cada colaborador.  
- Análises de People Analytics (headcount, desligamentos, evolução salarial) mais confiáveis ao longo do tempo.  
- Criação de um **processo replicável**, que pode ser reaplicado a novas extrações de dados com o mesmo layout.

---

## Fonte dos dados

O dataset foi disponibilizado pelo **Curso de Pós-Graduação em Análise de Dados e IA Voltada para Negócios** da Hashtag Treinamentos, em parceria com a Faculdade FNAT.  
A base contém dados fictícios de colaboradores com informações de cargo, área, salário, localização e histórico mensal, onde a chave principal é `ID_Funcionario`.

---

## Estrutura da base (colunas)

A base reconstruída possui as seguintes colunas:

| Coluna               | Tipo                | Descrição                                                                                                     |
|----------------------|---------------------|---------------------------------------------------------------------------------------------------------------|
| Data_Referencia      | Texto (dd/mm/aaaa)  | Primeiro dia do mês de referência (Jan/2019 a Dez/2024).                                                     |
| ID_Funcionario       | Inteiro             | Código único do funcionário (chave natural).                                                                  |
| Nome_Funcionario     | Texto               | Nome completo do funcionário.                                                                                 |
| Departamento         | Texto               | Departamento (Produção, TI, Administração, Vendas, Diretoria, Engenharia de Software).                        |
| Area                 | Texto               | Área de agrupamento (Operações, Tecnologia, Administrativo, Comercial, Executivo).                            |
| Cargo                | Texto               | Cargo ocupado (31 cargos distintos).                                                                          |
| Nivel                | Texto               | Nível hierárquico (Operacional, Especialista, Gerencial).                                                     |
| Estado               | Texto               | UF de lotação (SP, RJ, MG, BA).                                                                               |
| Tipo_Unidade         | Texto               | Classificação da unidade (Matriz = SP, Filial = RJ/MG/BA).                                                    |
| Salario              | Inteiro             | Salário mensal em reais (R$).                                                                                |
| Faixa_Salarial       | Texto               | Faixa salarial (0–5k, 5k–8k, 8k–12k, 12k–20k, 20k–35k, 35k–50k, 50k+).                                        |
| Data_Admissao        | Texto (dd/mm/aaaa)  | Data de admissão do funcionário.                                                                              |
| Data_Desligamento    | Texto (dd/mm/aaaa)  | Data de desligamento (vazio se funcionário ativo).                                                            |
| Status_Emprego       | Texto               | Status atual: Ativo, Desligamento voluntário ou Desligado por justa causa.                                   |
| Motivo_Desligamento  | Texto               | Motivo do desligamento (18 motivos distintos, ou `N/A - Ainda empregado`).                                    |
| Fonte_Recrutamento   | Texto               | Canal de recrutamento (LinkedIn, Indeed, Indicação de colaborador, etc.).                                     |
| Pontuacao_Desempenho | Texto               | Avaliação de desempenho (Acima do esperado, Atende plenamente, Precisa melhorar, Plano de melhoria).          |

---

## Metodologia (passo a passo)

Os passos implementados foram:

### 1. Importar bibliotecas e carregar dados

- Leitura da planilha `Base_People_Analytics.xlsx` em um DataFrame pandas (`df_original`).

### 2. Overview e detecção de inconsistências

- Inspeção de linhas, colunas, tipos, datas mínimas e máximas e amostras por colaborador.  
- Identificação de gaps e registros extras entre `Data_Admissao`, `Data_Desligamento` e `Data_Referencia`.

### 3. Construir uma tabela auxiliar por colaborador

- Seleção de `ID_Funcionario`, `Data_Admissao`, `Data_Desligamento` a partir da base original.  
- Preenchimento de `Data_Desligamento` vazia com a maior `Data_Referencia` disponível (colaborador ativo até a última competência da base).  
- Deduplicação por `ID_Funcionario` e ordenação crescente.

### 4. Calcular o número de meses por colaborador

- Cálculo de `Meses_Diferenca` como a diferença em meses entre `Data_Admissao` e `Data_Desligamento`, incluindo o mês final.  
- Exemplo: 2019-01-01 a 2024-12-01 = 72 meses.

### 5. Replicar linhas e gerar a nova Data_Referencia

- Uso de `repeat(Meses_Diferenca)` para replicar cada colaborador em tantas linhas quanto a quantidade de meses entre admissão e desligamento.  
- Criação de um contador `Num_Repeticao` (1, 2, 3, …) por `ID_Funcionario` com `groupby().cumcount() + 1`.  
- Cálculo da nova `Data_Referencia` adicionando `Num_Repeticao - 1` meses à `Data_Admissao` com `DateOffset(months=...)` e forçando sempre o primeiro dia do mês.

### 6. Resgatar Salario e Status_Emprego mês a mês

- A partir da base original, recuperação dos valores de `Salario` e `Status_Emprego` mais recentes para cada combinação (`ID_Funcionario`, `Data_Referencia`) criada.  
- Para datas em que não há registro original exato, usa-se a observação válida mais próxima anterior, garantindo coerência temporal.

### 7. Resgatar dimensões estáticas

- Seleção de colunas que não mudam no tempo para cada colaborador:  
  `Nome_Funcionario`, `Departamento`, `Area`, `Cargo`, `Nivel`, `Estado`, `Tipo_Unidade`, `Faixa_Salarial`, `Data_Admissao`, `Data_Desligamento`, `Motivo_Desligamento`, `Fonte_Recrutamento`, `Pontuacao_Desempenho`.  
- Ordenação por `ID_Funcionario` e `Data_Referencia` e uso da primeira ocorrência consistente para cada `ID_Funcionario`.  
- Merge dessa dimensão com a base reconstruída usando `ID_Funcionario` como chave.

### 8. Exportar a base final

- Geração da base final com 20.284 linhas e 17 colunas, contendo uma linha por colaborador por mês, com todas as informações preenchidas.  
- Exportação para Excel (`BasePeopleAnalytics_refeita.xlsx`) com todos os campos já ordenados e tipados.

---

## Tecnologias utilizadas

- **Linguagem:** Python 3.x.  
- **Bibliotecas:**  
  - `pandas` – manipulação e transformação da base tabular.  
  - `numpy` – apoio em operações numéricas.  
  - `pandas.tseries.offsets.DateOffset` – cálculo de deslocamentos mensais para construção de `Data_Referencia`.

---

## Como reproduzir

1. Clonar este repositório.  
2. Garantir que o arquivo da base original (`Base_People_Analytics.xlsx`) esteja na pasta correta.  
3. Instalar as dependências:

   ```bash
   pip install -r requirements.txt
   ```

4. Executar o notebook `Base_People_Analytics.ipynb` ou o script equivalente em `src/` (caso você tenha separado a lógica do notebook em um arquivo `.py`).