# Desafio de Transformação de Dados com Python e Dashboard com Looker

## Descrição do Projeto

Este projeto tem como objetivo transformar dados de uma planilha do Google Sheets em uma tabela detalhada por dia para cada vendedor, incluindo suas quotas e status mensais. O resultado é então visualizado em um dashboard no Looker.

## Estrutura do Projeto

- **Google Colab:** Script em Python para transformar os dados.
- **Google Sheets:** Planilhas contendo os dados de entrada e o resultado transformado.
- **Looker Dashboard:** Dashboard simples no Looker para visualizar as métricas.

## Funcionalidades

- **Transformação de Dados:** O script Python lê dados de entrada de uma planilha Google, aplica regras de transformação e gera uma tabela detalhada diária.
- **Visualização de Dados:** Dashboard no Looker mostrando as quotas mensais para cada vendedor e gerente.

## Instruções de Uso

### 1. Executar o Script Python no Google Colab

1. Abra o Google Colab e autentique sua conta do Google.
2. Instale a biblioteca `gspread` para manipular o Google Sheets.
3. Copie e cole o script Python fornecido (ver abaixo) em uma célula do Colab.
4. Execute o script para transformar os dados e exportá-los para uma nova planilha no Google Sheets.

### 2. Subir os Dados Transformados para o Google Sheets

1. O script Python exportará automaticamente os dados transformados para uma nova planilha no Google Sheets.
2. Verifique a planilha "Merged Data" para confirmar que os dados foram corretamente transformados.

### 3. Conectar o Google Sheets ao Looker

1. No Looker, configure uma conexão para o Google Sheets.
2. Importe os dados da planilha para o Looker criando uma nova view ou exploração baseada nos dados importados.

### 4. Criar o Dashboard no Looker

1. No Looker, crie uma nova exploração usando os dados importados.
2. Adicione visualizações para mostrar as métricas solicitadas (quota por mês para cada vendedor e gerente).
3. Salve e publique o dashboard.

## Estrutura do Script Python

O script Python realiza os seguintes passos:

1. **Autenticação no Google Sheets:** Utiliza a biblioteca `gspread` para autenticar e acessar as planilhas.
2. **Leitura dos Dados de Entrada:** Lê os dados das planilhas "Hiring", "Manager", "Roles" e "Targets".
3. **Transformação dos Dados:** Aplica regras de transformação para calcular status e quotas mensais para cada vendedor.
4. **Exportação dos Dados Transformados:** Exporta os dados transformados para uma nova planilha "Merged Data".


## Código Python para Transformação de Dados

O seguinte script em Python é usado para transformar dados de uma planilha Google Sheets e calcular as quotas de cada vendedor.

```python
import pandas as pd
from datetime import datetime
from google.colab import auth
import gspread
from google.auth import default
from gspread_dataframe import set_with_dataframe

# Instalar e autenticar Google Sheets
!pip install --upgrade gspread
auth.authenticate_user()

creds, _ = default()
gc = gspread.authorize(creds)

# Abrir a planilha e obter as folhas de trabalho
spreadsheet = gc.open('PythonTeste')
hiring_sheet = spreadsheet.worksheet('Hiring')
manager_sheet = spreadsheet.worksheet('Manager')
roles_sheet = spreadsheet.worksheet('Roles')
targets_sheet = spreadsheet.worksheet('Targets')

# Converter folhas de trabalho para DataFrames
hiring_df = pd.DataFrame(hiring_sheet.get_all_records())
manager_df = pd.DataFrame(manager_sheet.get_all_records())
roles_df = pd.DataFrame(roles_sheet.get_all_records())
targets_df = pd.DataFrame(targets_sheet.get_all_records())

# Verificar nomes das colunas do DataFrame manager_df
print("Colunas do manager_df:", manager_df.columns.tolist())

# Verificar nomes das colunas do DataFrame roles_df
print("Colunas do roles_df:", roles_df.columns.tolist())

# Mesclar os DataFrames, excluindo as colunas 'Dimension' e 'Value' de targets_df
merged_df = hiring_df.merge(manager_df[['Name', 'Manager', 'Manager Start Date', 'Manager End Date']], on='Name', how='left') \
                      .merge(roles_df[['Name', 'Role', 'Role Start Date', 'Role End Date']], on='Name', how='left') \
                      .merge(targets_df.drop(['Dimension', 'Value'], axis=1), on='Role', how='left')

# Remover linhas duplicadas com base em todas as colunas
merged_df.drop_duplicates(inplace=True)

# Criar uma nova planilha ou limpar uma existente
try:
    worksheet = spreadsheet.add_worksheet(title="Merged Data", rows="100", cols="20")
except gspread.exceptions.WorksheetNotFound:
    worksheet = spreadsheet.worksheet("Merged Data")
    worksheet.clear()

# Padronizar 'CSS Start Date' e 'CSS Attrition date' para o formato ISO8601 (YYYY-MM-DD)
merged_df['CSS Start Date'] = pd.to_datetime(merged_df['CSS Start Date'], errors='coerce').dt.strftime('%Y-%m-%d')
merged_df['CSS Attrition date'] = pd.to_datetime(merged_df['CSS Attrition date'], errors='coerce').dt.strftime('%Y-%m-%d')
merged_df['CSS Attrition date'].fillna('', inplace=True)  # Preencher NaN com string vazia

# Encontrar a data de início mais antiga
earliest_start_date = merged_df['CSS Start Date'].min()
start_date = datetime.strptime(earliest_start_date, '%Y-%m-%d')
end_date = datetime.now()
date_range = pd.date_range(start_date, end_date, freq='MS').strftime('%Y-%m-%d').tolist()

# Função para calcular status com base nas regras especificadas
def calculate_status_for_date(row, as_of_date):
    start_date_str = row['CSS Start Date']
    attrition_date_str = row.get('CSS Attrition date')
    start_date = datetime.strptime(start_date_str, '%Y-%m-%d')

    # Verifica se a data de saída (attrition date) está antes ou igual à data de referência
    if attrition_date_str and attrition_date_str != '' and datetime.strptime(attrition_date_str, '%Y-%m-%d') <= as_of_date:
        return 'Fired'

    # Verifica se a data de início é igual à data de referência
    if start_date == as_of_date:
        return 'Ramping 1'

    # Se a data de início é diferente da data de referência
    if start_date.month == as_of_date.month and start_date.year == as_of_date.year:
        return 'Ramping 0'

    # Calcula a diferença em meses entre a data de início e a data de referência
    months_difference = (as_of_date.year - start_date.year) * 12 + (as_of_date.month - start_date.month)

    # Avalia o status com base na diferença de meses
    if months_difference == 1:
        return 'Ramping 2' if start_date + pd.DateOffset(months=1) == as_of_date else 'Ramping 1'
    elif months_difference == 2:
        return 'Ramping 3' if start_date + pd.DateOffset(months=2) == as_of_date else 'Ramping 2'
    elif months_difference == 3:
        return 'Ramping 4' if start_date + pd.DateOffset(months=3) == as_of_date else 'Ramping 3'
    elif months_difference == 4:
        return 'Ramped' if start_date + pd.DateOffset(months=4) == as_of_date else 'Ramping 4'

    # Para meses após 'Ramped', o status deve ser 'Ramped'
    if months_difference > 4:
        return 'Ramped'

    # Caso padrão, se não houver correspondência
    return None

# Função para atribuir quota com base no papel e status
def assign_quota_based_on_status(role, status):
    quotas = {
        "Account Executive": {
            "Ramping 0": 0.00,
            "Ramping 1": 24.00,
            "Ramping 2": 24.00,
            "Ramping 3": 50.00,
            "Ramping 4": 75.00,
            "Ramped": 100.00
        },
        "Associate Account Executive": {
            "Ramping 0": 0.00,
            "Ramping 1": 24.00,
            "Ramping 2": 24.00,
            "Ramping 3": 50.00,
            "Ramped": 90.00
        }
    }
    return quotas.get(role, {}).get(status, 0)

# Criar um DataFrame para armazenar status e quotas para cada mês
status_quota_df = pd.DataFrame()

for date in date_range:
    as_of_date = datetime.strptime(date, '%Y-%m-%d')
    temp_df = merged_df.copy()
    temp_df['As_of_Date'] = date
    temp_df['Status'] = temp_df.apply(lambda row: calculate_status_for_date(row, as_of_date), axis=1)
    temp_df['Quotas_Pts'] = temp_df.apply(lambda row: assign_quota_based_on_status(row['Role'], row['Status']), axis=1)
    status_quota_df = pd.concat([status_quota_df, temp_df], ignore_index=True)

# Agrupar por Vendedor, Gerente, Status, Papel, Data de Início e Data de Saída, sumarizando as quotas
quota_per_month_df = status_quota_df.groupby(['As_of_Date', 'Name', 'CSS Start Date', 'CSS Attrition date', 'Manager', 'Manager Start Date', 'Manager End Date', 'Role', 'Role Start Date', 'Role End Date', 'Status'], as_index=False)['Quotas_Pts'].sum()

# Atualizar a planilha "Merged Data" com os dados de quotas por mês
set_with_dataframe(worksheet, quota_per_month_df) 

