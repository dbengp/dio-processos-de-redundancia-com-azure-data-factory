# dio-processos-de-redundancia-com-azure-data-factory
## Criando Processos de Redundância de Arquivos na Azure

#### Os conceitos e resumo foram retirados da leitura das documentações oficiais:
- <https://learn.microsoft.com/pt-br/azure/data-factory/>
- <https://learn.microsoft.com/pt-br/azure/data-factory/concepts-pipelines-activities>
- <https://learn.microsoft.com/pt-br/training/modules/use-data-factory-pipelines-fabric/>

### O Azure Data Factory (ADF): A Orquestração de Dados na Nuvem
- O Azure Data Factory e seus pipelines fornecem a espinha dorsal para uma arquitetura de dados moderna que serve às necessidades da Data Science. Eles garantem que os dados brutos de uma infinidade de fontes sejam tratados, transformados e entregues de forma confiável e escalável, permitindo que os cientistas de dados se concentrem em construir e refinar modelos preditivos que impulsionam o valor de negócio. O Azure Data Factory é um serviço ETL (Extract, Transform, Load) e de orquestração de dados baseado em nuvem, serverless e totalmente gerenciado, oferecido pela Microsoft Azure. Ele permite a criação de fluxos de trabalho orientados a dados para orquestrar e automatizar a movimentação e a transformação de dados em escala. No contexto de Data Science, o ADF atua como o "maestro" que organiza e executa as etapas necessárias para preparar os dados que alimentarão modelos de Machine Learning, análises preditivas e dashboards de Business Intelligence, garantindo que os cientistas de dados tenham acesso a dados limpos, consistentes e prontos para uso.

### Pipelines no ADF: Os Fluxos de Trabalho do Processamento de Dados
- Um pipeline no ADF é uma coleção lógica de atividades que juntas realizam uma tarefa. As atividades definem as ações a serem executadas nos dados, como mover dados, executar um script SQL, invocar um notebook Databricks, ou transformar dados usando um Data Flow. Como os Pipelines do ADF entregam valor no tratamento, transformação e processamento de dados:
- Tratamento de Dados (Injestão e Limpeza):
  * Conectividade Abrangente: Os pipelines do ADF podem se conectar a mais de 100 fontes de dados distintas, tanto on-premises quanto na nuvem (SQL Server, Azure SQL DB, Azure Synapse Analytics, Azure Blob Storage, Data Lake Storage, SAP, Oracle, Salesforce, APIs REST, etc.). Isso permite a ingestão de dados brutos de diversas origens que uma equipe de Data Science pode precisar.
  * Coleta Programada: A capacidade de agendar pipelines (por exemplo, a cada hora, diariamente) garante que os dados mais recentes sejam coletados e disponibilizados continuamente para análise e re-treinamento de modelos.
  * Detecção de Alterações (CDC): Conforme discutido, o ADF pode usar Change Data Capture para capturar apenas os dados novos ou modificados de bancos de dados transacionais, otimizando o processo de ingestão e mantendo os datasets de Data Science atualizados com eficiência.
  * Validação e Preparação Inicial: Atividades de pipeline podem incorporar lógica inicial para identificar e até mesmo corrigir inconsistências básicas, duplicatas ou valores nulos, preparando os dados para transformações mais complexas.
- Transformação e Processamento de Dados:
  * Mapping Data Flows: Este é o coração da capacidade de transformação do ADF. Permite que cientistas de dados e engenheiros de dados construam logicamente transformações de dados sem a necessidade de escrever código, utilizando uma interface visual arrastar-e-soltar. Isso é ideal para:
    * Agregação: Calcular totais, médias, contagens (como no cenário de transações por região).
    * Joins: Combinar dados de diferentes fontes para enriquecer datasets.
    * Filtering: Selecionar subconjuntos de dados relevantes para modelos específicos.
    * Derivação de Colunas: Criar novas features a partir de dados existentes, como cálculo de KPIs.
    * Limpeza e Enriquecimento: Manipulação de strings, tratamento de datas, padronização de formatos.
    * Particionamento: Organizar dados para otimizar o acesso e o desempenho de cargas de trabalho de Data Science.
  * Atividades Customizadas: Para transformações mais complexas ou cenários específicos, os pipelines podem orquestrar a execução de:
    * Notebooks Databricks: Para transformação de dados complexa usando PySpark, Scala, R.
    * Stored Procedures: Para lógica de transformação que reside no banco de dados.
    * Azure Functions/Web Activities: Para lógica customizada serverless.
    * Azure Batch/Azure Machine Learning Activities: Para processamento pesado ou execução de scripts de treinamento/scoring de modelos.
    * Escalabilidade e Desempenho: O ADF opera em uma arquitetura serverless, o que significa que ele gerencia automaticamente a escalabilidade e os recursos computacionais subjacentes para lidar com grandes volumes de dados de forma eficiente, sem que o usuário precise se preocupar com a infraestrutura.
- Entrega de Valor em uma Solução de Data Science:
  * Democratização dos Dados: Ao automatizar e padronizar o processo ETL, o ADF torna os dados limpos e transformados acessíveis aos cientistas de dados, reduzindo o tempo que eles gastam na "limpeza" e aumentando o tempo dedicado à modelagem e à extração de insights.
  * Operationalização de Modelos de ML:
    * Preparação de Dados para Treinamento: Pipelines do ADF podem preparar e entregar dados em larga escala para o treinamento de modelos de Machine Learning (por exemplo, em Azure Machine Learning ou Databricks).
    * Engenharia de Features: ADF pode ser usado para criar e atualizar features para modelos de ML, garantindo que as features usadas no treinamento e na inferência sejam consistentes.
    * Inferência Batch: Após o treinamento, o ADF pode orquestrar a execução de pipelines de inferência (scoring) em lote, onde os modelos de ML são aplicados a novos dados para gerar previsões em escala.
    * Feedback Loops: Os pipelines podem coletar os resultados da inferência e feedback para re-treinamento contínuo de modelos, fechando o ciclo de vida do ML.
  * Governança e Monitoramento: O ADF oferece capacidades robustas de monitoramento que permitem rastrear o progresso dos pipelines, identificar gargalos, solucionar erros e garantir a qualidade dos dados entregues para as soluções de Data Science. Isso é crucial para manter a integridade e a confiança nos dados e nos modelos.
  * Integração com o Ecossistema Azure: A integração nativa com outros serviços Azure (Azure Synapse Analytics, Azure Databricks, Azure Machine Learning, Power BI) permite que o ADF atue como um hub central para toda a sua plataforma de dados e AI.
  * Custos Otimizados: Por ser serverless, o ADF cobra apenas pelo tempo de execução das atividades, o que o torna uma solução custo-efetiva, especialmente para cargas de trabalho que não são contínuas 24/7.

### Cenário desse projeto feito com CLI
#### Processos de Redundância de Arquivos na Azure dentro de um pipeline que tem como fonte de dados uma instância do SQL Server com um banco de dados de transaçãoes em uma rede de e-commerce global, de onde irá se salvar totais dessas transações em um storage account separando-os por região de modo decrescente de volume de transações.
- Pré-requisitos:
  * Azure CLI instalado e logado (az login).
  * Uma assinatura Azure ativa.
- Começo com algumas varáveis de ambiente necessárias:
```
# Informações básicas
RESOURCE_GROUP_NAME="rg-ecom-data-redundancy"
LOCATION="eastus" # Escolha a região mais próxima ou desejada

# Configurações do SQL Server
SQL_SERVER_NAME="sqlserver-ecom-global-001"
SQL_DB_NAME="db-ecom-transactions"
SQL_ADMIN_USER="sqladmin" # Usuário administrador do SQL
SQL_ADMIN_PASSWORD="YourStrongPassword123!" # Senha forte! Use Azure Key Vault em produção!

# Configurações do Storage Account
STORAGE_ACCOUNT_NAME="stecomglobalredundancy01" # Deve ser globalmente único e em minúsculas
STORAGE_CONTAINER_NAME="transactions-by-region"

# Configurações do Data Factory
DATA_FACTORY_NAME="adf-ecom-redundancy-001"

# Caminhos para arquivos de definição JSON (conceituais - você criaria esses arquivos)
# No ambiente real, esses JSONs seriam gerados pelo ADF Studio ou por seu processo de CI/CD
LINKED_SERVICE_SQL_JSON_FILE="linkedServiceSqlServer.json"
LINKED_SERVICE_STORAGE_JSON_FILE="linkedServiceAzureBlobStorage.json"
DATASET_SQL_JSON_FILE="datasetSqlServerTransactions.json"
DATASET_STORAGE_JSON_FILE="datasetStorageOutput.json"
DATA_FLOW_JSON_FILE="dataFlowTransactionsByRegion.json" # Para totais, agrupamento e ordenação
PIPELINE_JSON_FILE="pipelineProcessTransactions.json" 
```
- Criação do Grupo de Recursos:
```
echo "Criando o Grupo de Recursos: $RESOURCE_GROUP_NAME..."
az group create \
  --name $RESOURCE_GROUP_NAME \
  --location $LOCATION
echo "Grupo de Recursos '$RESOURCE_GROUP_NAME' criado com sucesso."
```
- Criação do Azure SQL Server e Banco de Dados
```
echo "Criando o Azure SQL Server: $SQL_SERVER_NAME..."
az sql server create \
  --name $SQL_SERVER_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --location $LOCATION \
  --admin-user $SQL_ADMIN_USER \
  --admin-password $SQL_ADMIN_PASSWORD
echo "Azure SQL Server '$SQL_SERVER_NAME' criado com sucesso."

echo "Criando o Banco de Dados SQL: $SQL_DB_NAME..."
az sql db create \
  --name $SQL_DB_NAME \
  --server $SQL_SERVER_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --edition Basic \
  --family Gen5 \
  --capacity 20 # Exemplo: 20 GB de armazenamento
echo "Banco de Dados SQL '$SQL_DB_NAME' criado com sucesso."

echo "Configurando regra de firewall para permitir acesso de serviços Azure ao SQL Server..."
az sql server firewall-rule create \
  --resource-group $RESOURCE_GROUP_NAME \
  --server $SQL_SERVER_NAME \
  --name "AllowAllWindowsAzureIps" \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
echo "Regra de firewall configurada."
```
- Criação do Azure Storage Account
```
echo "Criando o Storage Account: $STORAGE_ACCOUNT_NAME..."
az storage account create \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
echo "Storage Account '$STORAGE_ACCOUNT_NAME' criado com sucesso."

# Obter a string de conexão para uso em Linked Service (apenas para demonstração)
STORAGE_CONN_STRING=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --query 'connectionString' \
  --output tsv)
echo "String de Conexão do Storage Account (para Linked Service): $STORAGE_CONN_STRING"

echo "Criando contêiner '$STORAGE_CONTAINER_NAME' no Storage Account..."
az storage container create \
  --name $STORAGE_CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT_NAME \
  --public-access off # Private by default
echo "Contêiner '$STORAGE_CONTAINER_NAME' criado com sucesso."
```
- Criação do Azure Data Factory
```
echo "Criando o Azure Data Factory: $DATA_FACTORY_NAME..."
az datafactory create \
  --name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --location $LOCATION
echo "Azure Data Factory '$DATA_FACTORY_NAME' criado com sucesso."
```
- Definição e Implantação dos Recursos do Data Factory (Conceitual)
  * Os JSONs das definições: Use a interface do usuário do Azure Data Factory Studio -> seu Data Factory -> Open Azure Data Factory Studio para criar o Linked Service para o SQL Server, o LinkedService para o Storage Account, os Datasets (entrada e saída), o Data Flow de Mapeamento (para a lógica de agregação, agrupamento por região e ordenação) e, finalmente, o Pipeline que orquestrará tudo.
  * Exporte as definições JSON: Após criar os componentes, você pode exportar suas definições como arquivos JSON. Em um ambiente de CI/CD, essas definições JSON estariam no seu repositório de código.
- Exemplos de Estrutura de Conteúdo JSON para efeito de demonstração:
  * linkedServiceSqlServer.json
```
{
    "name": "SqlServerLinkedService",
    "properties": {
        "annotations": [],
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "integrated security=False;data source=<seu_servidor_sql>.database.windows.net;initial catalog=<seu_banco_de_dados>;user id=<seu_usuario>;password=<sua_senha>",
            "encryptedCredential": "..."
        },
        "connectVia": {
            "referenceName": "AutoResolveIntegrationRuntime",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```
  * linkedServiceAzureBlobStorage.json
```
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "annotations": [],
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<seu_storage_account>;AccountKey=<sua_chave_de_conta_storage>;EndpointSuffix=core.windows.net"
        },
        "connectVia": {
            "referenceName": "AutoResolveIntegrationRuntime",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```
  * datasetSqlServerTransactions.json
```
{
    "name": "SqlServerTransactionsDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "SqlServerLinkedService",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "SqlServerTable",
        "schema": [],
        "typeProperties": {
            "tableName": "Transactions"
        }
    }
}
``` 
  * datasetStorageOutput.json - CSV com partição dinâmica por região
```
{
    "name": "StorageOutputDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "folderPath": "@dataset().outputFolder",
                "fileName": "transactions_totals.csv",
                "container": "transactions-by-region"
            },
            "columnDelimiter": ",",
            "escapeChar": "\\",
            "quoteChar": "\""
        },
        "schema": []
    },
    "parameters": {
        "outputFolder": {
            "type": "String"
        }
    }
}
```
  * dataFlowTransactionsByRegion.json
```
{
    "name": "DataFlowTransactionsByRegion",
    "properties": {
        "type": "MappingDataFlow",
        "typeProperties": {
            "sources": [
                {
                    "dataset": {
                        "referenceName": "SqlServerTransactionsDataset",
                        "type": "DatasetReference"
                    },
                    "name": "sourceTransactions"
                }
            ],
            "transformations": [
                {
                    "name": "aggregateTotals",
                    "type": "Aggregate",
                    "description": "Calculates total transactions per region"
                },
                {
                    "name": "sortTotals",
                    "type": "Sort",
                    "description": "Sorts by total volume descending"
                }
                // ... outras transformações para extrair região, etc.
            ],
            "sinks": [
                {
                    "dataset": {
                        "referenceName": "StorageOutputDataset",
                        "type": "DatasetReference"
                    },
                    "name": "sinkOutput"
                }
            ],
            "script": "source(entity='Transactions', allowSchemaDrift: true, validateSchema: false, isolationLevel: 'ReadUncommitted', partitionBy: 'none') ~> sourceTransactions\nsourceTransactions aggregate(groupBy(region), totalTransactions = sum(transactionAmount)) ~> aggregateTotals\naggregateTotals sort(desc(totalTransactions)) ~> sortTotals\nsortTotals sink(allowSchemaDrift: true,\n\tvalidateSchema: false,\n\tpartitionFileNames:['transactions_by_region.csv'],\n\tpartitionBy('region'),\n\tmapColumn(region,\n\t\ttotalTransactions)\n\t) ~> sinkOutput"
        }
    }
}
```
  * pipelineProcessTransactions.json
```
{
    "name": "ProcessTransactionsPipeline",
    "properties": {
        "activities": [
            {
                "name": "RunDataFlowToProcessTransactions",
                "type": "ExecuteDataFlow",
                "dependsOn": [],
                "policy": {
                    "timeout": "0.12:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "dataFlow": {
                        "referenceName": "DataFlowTransactionsByRegion",
                        "type": "DataFlowReference"
                    },
                    "compute": {
                        "coreCount": 8,
                        "clusterType": "General"
                    },
                    "traceLevel": "Fine"
                },
                "linkedServiceName": {
                    "referenceName": "AzureBlobStorageLinkedService",
                    "type": "LinkedServiceReference"
                }
            }
        ],
        "folder": {
            "name": "E-commerce Redundancy"
        },
        "annotations": [],
        "lastPublishTime": "2025-05-30T12:00:00Z"
    }
}
```
- Comandos CLI para Carregar as Definições no Data Factory:
```
echo "Carregando Linked Service para SQL Server no Data Factory..."
az datafactory linked-service create \
  --name "SqlServerLinkedService" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$LINKED_SERVICE_SQL_JSON_FILE"
echo "Linked Service SQL Server carregado."

echo "Carregando Linked Service para Azure Blob Storage no Data Factory..."
az datafactory linked-service create \
  --name "AzureBlobStorageLinkedService" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$LINKED_SERVICE_STORAGE_JSON_FILE"
echo "Linked Service Azure Blob Storage carregado."

echo "Carregando Dataset para SQL Server no Data Factory..."
az datafactory dataset create \
  --name "SqlServerTransactionsDataset" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$DATASET_SQL_JSON_FILE"
echo "Dataset SQL Server carregado."

echo "Carregando Dataset de Saída para Storage no Data Factory..."
az datafactory dataset create \
  --name "StorageOutputDataset" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$DATASET_STORAGE_JSON_FILE"
echo "Dataset de Saída carregado."

echo "Carregando Data Flow de Mapeamento no Data Factory..."
az datafactory data-flow create \
  --name "DataFlowTransactionsByRegion" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$DATA_FLOW_JSON_FILE"
echo "Data Flow carregado."

echo "Carregando Pipeline no Data Factory..."
az datafactory pipeline create \
  --name "ProcessTransactionsPipeline" \
  --factory-name $DATA_FACTORY_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --properties "@$PIPELINE_JSON_FILE"
echo "Pipeline carregado."

echo "Publicando todas as alterações no Data Factory..."
# Ao criar os recursos via CLI/API, eles são automaticamente 'publicados' no serviço.
# Se você estiver fazendo alterações em um ADF existente, usaria 'az datafactory pipeline update', etc.
echo "Os recursos são criados/atualizados diretamente no Data Factory ao usar os comandos 'create' acima."
echo "Para executar o pipeline, você pode usar:"
echo "az datafactory pipeline create-run --name ProcessTransactionsPipeline --factory-name $DATA_FACTORY_NAME --resource-group $RESOURCE_GROUP_NAME"
```
