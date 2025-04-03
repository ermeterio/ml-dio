📌 1. Criação do Conjunto de Dados

🔹 Geração do CSV
    O primeiro passo é criar um arquivo CSV contendo os dados históricos de vendas. Este arquivo será utilizado para treinar o modelo de previsão.

🔹 Criação do Arquivo ML Table
    O arquivo CSV precisa ser convertido em um formato ML Table, que é compatível com o Azure ML.
  
🚀 2. Configuração do Job de Treinamento
🔹 Configurações Básicas

    Workspace utilizado: MLErmeterio
    Tipo: Train automatically
    Nome do Job: sales_prevision
    Descrição: Automated machine learning for sales prevision

🔹 Definição da Tarefa e Dados

    Tipo de Tarefa: Regression

📂 Criando o Dataset

    Nome: sales-prevision
    Descrição: Historic sales prevision data
    Tipo: Table (mltable)

🔹 Fonte de Dados

    Origem: From local files

🔹 Armazenamento de Destino

    Tipo de Datastore: Azure Blob Storage
    Nome: workspaceblobstore

⚙️ 3. Configurações da Tarefa

🔹 Parâmetros da Tarefa

    Tipo de Tarefa: Regression
    Dataset: salesPrevision
    Coluna Alvo: vendas (decimal)

🔹 Configurações Adicionais

    Métrica Primária: NormalizedRootMeanSquaredError
    Explicar melhor modelo: ❌ Não selecionado
    Habilitar ensemble stacking: ❌ Não selecionado
    Utilizar todos os modelos suportados: ❌ Não selecionado
    Modelos permitidos: ✅ RandomForest e LightGBM

🔧 4. Definição de Limites

    Máximo de testes: 3
    Máximo de testes concorrentes: 3
    Máximo de nós: 3
    Limite de pontuação métrica: 0.085 (Se um modelo atingir essa métrica, o job é encerrado.)
    Tempo limite da experiência: 15 minutos
    Tempo limite por iteração: 15 minutos
    Habilitar término antecipado: ✅ Selecionado

💻 5. Configuração de Computação

    Tipo de Computação: Serverless
    Tipo de Máquina Virtual: CPU
    Nível da Máquina Virtual: Dedicated
    Tamanho da Máquina Virtual: Standard_DS3_V2*
    Número de Instâncias: 1
