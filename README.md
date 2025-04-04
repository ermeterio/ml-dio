# 📌 Guia: Criação de um Novo Modelo de Previsão

Este guia descreve os passos necessários para configurar um modelo de previsão de vendas utilizando **Azure Machine Learning**.

---

## 📂 1. Criação do Conjunto de Dados

### 🔹 Geração do CSV
O primeiro passo é criar um arquivo CSV contendo os dados históricos de vendas. Este arquivo será utilizado para treinar o modelo de previsão.

#### **📌 Exemplo de Estrutura do CSV**

| gastos_marketing | sazonalidade | descontos | vendas |
|-----------------|--------------|----------|--------|
| 5000           | 1            | 10       | 9200   |
| 7000           | 0            | 15       | 8500   |
| 3000           | 1            | 5        | 7800   |
| 9000           | 1            | 20       | 11200  |

### 🔹 Criação do Arquivo ML Table
O arquivo CSV precisa ser convertido em um formato **ML Table**, compatível com o Azure ML.

---

## 🚀 2. Configuração do Job de Treinamento

### 🔹 Configurações Básicas
- **Workspace utilizado**: `MLErmeterio`
- **Tipo**: `Train automatically`
- **Nome do Job**: `sales_prevision`
- **Descrição**: `Automated machine learning for sales prevision`

### 🔹 Definição da Tarefa e Dados
- **Tipo de Tarefa**: `Regression`

#### 📂 Criando o Dataset
- **Nome**: `sales-prevision`
- **Descrição**: `Historic sales prevision data`
- **Tipo**: `Table (mltable)`

#### 🔹 Fonte de Dados
- **Origem**: `From local files`

#### 🔹 Armazenamento de Destino
- **Tipo de Datastore**: `Azure Blob Storage`
- **Nome**: `workspaceblobstore`

---

## ⚙️ 3. Configurações da Tarefa

### 🔹 Parâmetros da Tarefa
- **Tipo de Tarefa**: `Regression`
- **Dataset**: `bike-rentals`
- **Coluna Alvo**: `rentals (integer)`

### 🔹 Configurações Adicionais
- **Métrica Primária**: `NormalizedRootMeanSquaredError`
- **Explicar melhor modelo**: ❌ `Não selecionado`
- **Habilitar ensemble stacking**: ❌ `Não selecionado`
- **Utilizar todos os modelos suportados**: ❌ `Não selecionado`
- **Modelos permitidos**: ✅ `RandomForest` e `LightGBM`

---

## 🔧 4. Definição de Limites

- **Máximo de testes**: `3`
- **Máximo de testes concorrentes**: `3`
- **Máximo de nós**: `3`
- **Limite de pontuação métrica**: `0.085` (Se um modelo atingir essa métrica, o job é encerrado.)
- **Tempo limite da experiência**: `15 minutos`
- **Tempo limite por iteração**: `15 minutos`
- **Habilitar término antecipado**: ✅ `Selecionado`

---

## 💻 5. Configuração de Computação

- **Tipo de Computação**: `Serverless`
- **Tipo de Máquina Virtual**: `CPU`
- **Nível da Máquina Virtual**: `Dedicated`
- **Tamanho da Máquina Virtual**: `Standard_DS3_V2*`
- **Número de Instâncias**: `1`

---

## 🧪 6. Testes

Após a implantação do modelo, podemos testar o endpoint utilizando JSON para enviar os dados ao modelo.

### 🔹 **Formato do JSON de Teste**

```json
{
  "input_data": {
    "columns": [
      "gastos_marketing",
      "sazonalidade",
      "descontos"
    ],
    "index": [0],
    "data": [[5000, 1, 10]]
  }
}
```

### 🔹 **Chamando o Endpoint no C#**

```c#
// .NET Framework 4.7.1 or greater must be used

using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

namespace CallRequestResponseService
{
    class Program
    {
        static void Main(string[] args)
        {
            InvokeRequestResponseService().Wait();
        }

        static async Task InvokeRequestResponseService()
        {
            var handler = new HttpClientHandler()
            {
                ClientCertificateOptions = ClientCertificateOption.Manual,
                ServerCertificateCustomValidationCallback =
                        (httpRequestMessage, cert, cetChain, policyErrors) => { return true; }
            };
            using (var client = new HttpClient(handler))
            {
                // Request data goes here
                // The example below assumes JSON formatting which may be updated
                // depending on the format your endpoint expects.
                // More information can be found here:
                // https://docs.microsoft.com/azure/machine-learning/how-to-deploy-advanced-entry-script
                var requestBody = @"{}";
                
                // Replace this with the primary/secondary key, AMLToken, or Microsoft Entra ID token for the endpoint
                const string apiKey = "";
                if (string.IsNullOrEmpty(apiKey))
                {
                    throw new Exception("A key should be provided to invoke the endpoint");
                }
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue( "Bearer", apiKey);
                client.BaseAddress = new Uri("https://mlermeterio-wlrqy.eastus2.inference.ml.azure.com/score");

                var content = new StringContent(requestBody);
                client.DefaultRequestHeaders.Add("Accept", "application/json");
                content.Headers.ContentType = new MediaTypeHeaderValue("application/json");

                // WARNING: The 'await' statement below can result in a deadlock
                // if you are calling this code from the UI thread of an ASP.Net application.
                // One way to address this would be to call ConfigureAwait(false)
                // so that the execution does not attempt to resume on the original context.
                // For instance, replace code such as:
                //      result = await DoSomeTask()
                // with the following:
                //      result = await DoSomeTask().ConfigureAwait(false)
                HttpResponseMessage response = await client.PostAsync("", content);

                if (response.IsSuccessStatusCode)
                {
                    string result = await response.Content.ReadAsStringAsync();
                    Console.WriteLine("Result: {0}", result);
                }
                else
                {
                    Console.WriteLine(string.Format("The request failed with status code: {0}", response.StatusCode));

                    // Print the headers - they include the requert ID and the timestamp,
                    // which are useful for debugging the failure
                    Console.WriteLine(response.Headers.ToString());

                    string responseContent = await response.Content.ReadAsStringAsync();
                    Console.WriteLine(responseContent);
                }
            }
        }
    }
}

```

---
