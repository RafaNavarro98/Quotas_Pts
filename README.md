# Desafio de Transformação de Dados e Dashboard com Looker

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



   ```sh
   git clone https://github.com/seu-usuario/data-transformation-project.git
   cd data-transformation-project
pip install -r requirements.txt



