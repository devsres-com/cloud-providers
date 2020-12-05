+++
title = "AWS e Azure: Armazenamento de objetos"
description = "'Tradução' dos nomes entre os cloud providers"
author = "Marcelo Andrade"
date = "2020-12-04"
tags = ["storage","associate","az900"]
categories = ["computing","associate", "az900"]
[[images]]
  src = "img/main/blobvss3.png"
  alt = "Azure Blob vs S3"
  stretch = ""
+++

Se você conhece bastante de um Cloud Provider, acredito que o primeiro passo para estudar um segundo seria entender o básico da equivalência entre eles, meio que um "dicionário" para traduzir entre dois conjuntos distintos de vocabulários. 

Sempre existirão diferenças conceituais estruturantes, em especial nos Cloud Providers que vieram depois e puderam arranjar, de maneira mais elegante, a apresentação dos seus serviços. 

No fim do dia, uma máquina virtual é uma máquina virtual. Se você chama o disco de *Elastic Block Storage volume* ou *Azure Disk*, ele segue sendo um disco de uma máquina virtual.

Abaixo seguem alguns tópicos relevantes de mapeamento de um para o outro.

||Azure Blob|AWS S3|Observações|
|-------|---------|---------|-------|
|Arquivos|Blob|Objeto||
|Pastas|-|-|As 'pastas' não existem, são um recurso visual derivando as '/' dos nomes|
|Raiz do armazenamento|Container|Bucket|O nome do Bucket na AWS é global; usuários não podem repetir nomes de buckets que já existem;|
|Hierarquia adicional|Storage Account|-|O nome da Storage Account é global; usuários não podem repetir nomes de Storage Accounts que já existem;|
|Hierarquia adicional|Resource Group|-|Nada existe na Azure sem pertencer a um Resource Group.|

Logo, para a Azure, arquivos são **Blobs** armazenados em **Containers** que pertencem a uma **Storage Account** de um **Resource Group**. Já para a AWS, arquivos são **objetos** armazenados em **Buckets** do serviço S3. 

A [própria Microsoft](https://docs.microsoft.com/en-us/azure/architecture/aws-professional/storage) postou uma comparação sobre as diferenças, mas pouco relevante em explorar a terminologia.

||Azure Blob|AWS S3|Observações|
|-------|---------|---------|-------|
|Tamanho máximo de arquivos|Variável de acordo com o tipo. Padrão é 190TB |5 TB|[Diferentes tipos de blobs na Azure](https://docs.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)|
|Classes|Hot, Cool, Archive|Standard,Infrequent Access, os mesmos mas com Redundância reduzida (One Zone), Reduced Redundancy, Glacier, Glacier Deep Archive|
|Classes|Local (LRS), Zone (ZRS|S3 One Zone, Standard|
|Classes|Geo (GRS), Geozone (GZRS)|S3IA/S3 com Cross Region Replication |GZRS faz replicação entre zonas de uma mesma Region|
|Classes|Cool Blob Storage|Infrequent Access|
|Classes|Storage Archive Blob Storage|Glacier|
|Integração com CDN|Container $web de uma Storage Account com o recurso habilitado|Bucket com configuração especializada.|



## Bônus: linha de comando

Tudo bem que a *aws cli* é muito pouco intuitiva para a trabalhar com buckets, mas ainda assim, é bem simples.

```
# # mb de 'make bucket' :O
# aws s3 mb s3://dev-sres-first-bucket
make_bucket: dev-sres-first-bucket

# aws s3 ls
2020-12-04 23:42:42 dev-sres-first-bucket 
```

Upload de arquivos: também tranquilo:

```
# aws s3 cp teste  s3://dev-sres-first-bucket
upload: ./teste to s3://dev-sres-first-bucket/teste

# aws s3 ls  s3://dev-sres-first-bucket
2020-12-04 23:44:46          6 teste
```

E na Azure?

Você precisa:

* criar um 'resource group':
```
# az group create --name so-queria-criar-um-bucket
RequiredArgumentMissingError: the following arguments are required: --location/-l
Try this: 'az group create -l westus -n MyResourceGroup'

# tá bom, dá pra configurar a zona default para ele não ficar pedindo; eu já havia feito no AWS cli, então desconsidere:
# az configure --defaults location=eastus
{
  "id": "/subscriptions/f2b2b75e-5917-4ab1-9efd-d8fd48095700/resourceGroups/so-queria-criar-um-bucket",
  "location": "eastus",
  "managedBy": null,
  "name": "so-queria-criar-um-bucket",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}

# verbose much, huh?
```
* Criar um 'storage account':
```
# az storage account create -n por-favor-so-um-bucket -g so-queria-criar-um-bucket
BadRequestError: (AccountNameInvalid) por-favor-so-queria-um-bucket is not a valid storage account name. Storage account name must be between 3 and 24 characters in length and use numbers and lower-case letters only.

# # NA AWS O LIMITE É 63 CARACTERES E DEIXA HIFENS!!!!!!
 az storage account create -n prfvrmeubucket -g so-queria-criar-um-bucket
 ... 2 min depois ...
 {- Finished ..
  "accessTier": "Hot",
  "allowBlobPublicAccess": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2020-12-05T03:02:26.090589+00:00",
  "customDomain": null,
  "enableHttpsTrafficOnly": true,
  "encryption": {
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": null,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2020-12-05T03:02:26.184345+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2020-12-05T03:02:26.184345+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/f2b2b75e-5917-4ab1-9efd-d8fd48095700/resourceGroups/so-queria-criar-um-bucket/providers/Microsoft.Storage/storageAccounts/prfvrmeubucket",
  "identity": null,
  "isHnsEnabled": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "eastus",
  "minimumTlsVersion": null,
  "name": "prfvrmeubucket",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://prfvrmeubucket.blob.core.windows.net/",
    "dfs": "https://prfvrmeubucket.dfs.core.windows.net/",
    "file": "https://prfvrmeubucket.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://prfvrmeubucket.queue.core.windows.net/",
    "table": "https://prfvrmeubucket.table.core.windows.net/",
    "web": "https://prfvrmeubucket.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "resourceGroup": "so-queria-criar-um-bucket",
  "routingPreference": null,
  "secondaryEndpoints": {
    "blob": "https://prfvrmeubucket-secondary.blob.core.windows.net/",
    "dfs": "https://prfvrmeubucket-secondary.dfs.core.windows.net/",
    "file": null,
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://prfvrmeubucket-secondary.queue.core.windows.net/",
    "table": "https://prfvrmeubucket-secondary.table.core.windows.net/",
    "web": "https://prfvrmeubucket-secondary.z13.web.core.windows.net/"
  },
  "secondaryLocation": "westus",
  "sku": {
    "name": "Standard_RAGRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": "available",
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}

# # VERBOSE MUCH ?
```
* Finalmente, criar um container:
```
# az storage container create --account-name prfvrmeubucket --name plmorddeus-um-bucket
There are no credentials provided in your command and environment, we will query for the account key inside your storage account.
Please provide --connection-string, --account-key or --sas-token as credentials, or use `--auth-mode login` if you have required RBAC roles in your command. For more information about RBAC roles in storage, visit https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-cli.
Setting the corresponding environment variables can avoid inputting credentials in your command. Please use --help to get more information.
{
  "created": true
}

# # Que alerta esquisito!
```
Agora, podemos fazer o upload!

Vamos fazer certo, ele falou que eu devo especificar --auth-mode login para remover a mensagem de erro:

```
# az storage blob upload --account-name prfvrmeubucket --container-name plmorddeus-um-bucket --name yay --file teste --auth-mode login
UnknownError:
You do not have the required permissions needed to perform this operation.
Depending on your operation, you may need to be assigned one of the following roles:
    "Storage Blob Data Contributor"
    "Storage Blob Data Reader"
    "Storage Queue Data Contributor"
    "Storage Queue Data Reader"

If you want to use the old authentication method and allow querying for the right account key, please use the "--auth-mode" parameter and "key" value.

# # nay
```

Para você fazer do jeito certo, você precisa atribuir a função de 'colaborador de dados do blob de armazenamento' para o seu usuário que já é administrador. Porque né, é importante:

```
# # Uma linha tranquila e trivial.
# az ad signed-in-user show --query objectId -o tsv | az role assignment create --query objectId -o tsv --role "Storage Blob Data Contributor"  --assignee @- --scope "/subscriptions/f2b2b75e-5917-4ab1-9efd-d8fd48
095700/resourceGroups/so-queria-criar-um-bucket/providers/Microsoft.Storage/storageAccounts/prfvrmeubucket"
```

Será que agora vai?

```
# az storage blob upload --account-name prfvrmeubucket --container-name plmorddeus-um-bucket --name yay --file teste --auth-mode login
UnknownError:
You do not have the required permissions needed to perform this operation.
Depending on your operation, you may need to be assigned one of the following roles:
    "Storage Blob Data Contributor"
    "Storage Blob Data Reader"
    "Storage Queue Data Contributor"
    "Storage Queue Data Reader"

If you want to use the old authentication method and allow querying for the right account key, please use the "--auth-mode" parameter and "key" value.
```

Pô, mas eu fiz certo! 

(...Muitos minutos depois olhando a interface e tentando descobrir o que deu errado...)

Deixa eu ler a mensagem de erro de novo:

```
# az storage blob upload --account-name prfvrmeubucket --container-name plmorddeus-um-bucket --name yay --file teste --auth-mode login
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D898D09C5C2921\"",
  "lastModified": "2020-12-05T03:48:21+00:00"
}
```

Hum?

Azure Cloud, ladies and gentlemen!