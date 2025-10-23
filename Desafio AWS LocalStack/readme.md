## Desafio AWS LocalStack – Automação com Lambda e S3

Este laboratório tem como objetivo consolidar os conhecimentos em tarefas automatizadas utilizando AWS Lambda, S3 e DynamoDB, simulados por meio da ferramenta LocalStack.

A proposta é reproduzir um ambiente local que simula os serviços da AWS, permitindo testar automações e integrações entre funções Lambda, buckets S3 e tabelas DynamoDB sem precisar de uma conta AWS real.

## Objetivos do Desafio

Criar uma arquitetura em que um usuário sobe um arquivo de Notas Fiscais para um S3 que aciona uma trigger para o lambda gravar os dados em um banco de dados

## Tecnologias Utilizadas

LocalStack – Simulador local dos serviços AWS

PowerShell – Para execução dos comandos locais

Python – Para geração do arquivo JSON fake

AWS CLI – Para criação e gerenciamento dos recursos simulados

NoSQL Workbench – Para visualização e consulta no DynamoDB

## Insights sobre a Ferramenta LocalStack

O LocalStack é uma ferramenta poderosa que emula localmente os serviços da AWS.
Ele é especialmente útil para desenvolvedores que desejam testar automações, scripts e integrações sem custo, sem risco e com total controle do ambiente.

Principais benefícios:

Evita custos com a AWS real

Permite testes rápidos e offline

Simula serviços como S3, Lambda, DynamoDB, SQS, entre outros

Facilita a automação de pipelines locais e o desenvolvimento de IaC (Infrastructure as Code)

## Passo a Passo da Prática
1️⃣ Acessar o LocalStack

Entre no site oficial: https://localstack.cloud

Vá até Get Started → Download

Siga as instruções para instalação no seu sistema operacional.

2️⃣ Verificar a instalação

No PowerShell, execute:

localstack --version


Se instalado corretamente, será exibida a versão atual do LocalStack.

3️⃣ Iniciar o LocalStack em background

Utilize o comando disponibilizado na documentação (exemplo):

localstack start -d


O parâmetro -d (ou --detach) faz com que o serviço rode em segundo plano.

4️⃣ Criar a Função Lambda
aws lambda create-function --function-name ProcessarNotasFiscais --runtime python3.9 --role arn:aws:iam::000000000000:role/lambda-role --handler grava_db.lambda_handler --zip-file fileb://lambda_function.zip --endpoint-url=http://localhost:4566


Verificar se a função Lambda foi criada:

aws lambda list-functions --endpoint-url=http://localhost:4566

5️⃣ Criar o bucket S3
awslocal s3api create-bucket --bucket notas-fiscais-upload


Conceder permissão ao S3 para invocar a Lambda:

aws lambda add-permission --function-name ProcessarNotasFiscais --statement-id s3-trigger-permission --action "lambda:InvokeFunction" --principal s3.amazonaws.com --source-arn "arn:aws:s3:::notas-fiscais-upload" --endpoint-url=http://localhost:4566


Configurar a notificação no bucket S3:

aws s3api put-bucket-notification-configuration --bucket notas-fiscais-upload --notification-configuration file://notification_roles.json --endpoint-url=http://localhost:4566


Validar a configuração da notificação:

aws s3api get-bucket-notification-configuration --bucket notas-fiscais-upload --endpoint-url=http://localhost:4566

6️⃣ Criar a tabela DynamoDB
aws dynamodb create-table --endpoint-url=http://localhost:4566 --table-name NotasFiscais --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5


Verificar a tabela:

aws dynamodb list-tables --endpoint-url=http://localhost:4566

7️⃣ (Opcional) Instalar o NoSQL Workbench

Baixe o NoSQL Workbench for DynamoDB (ou outro cliente de sua preferência):
🔗 Documentação oficial

8️⃣ Gerar arquivo de teste JSON

Crie e execute o script Python para gerar o arquivo notas_fiscais.json:

python gerar_dados.py


O script criará um arquivo fake com dados de notas fiscais para o teste.

9️⃣ Enviar o arquivo para o S3
aws s3 cp notas_fiscais_2025.json s3://notas-fiscais-upload --endpoint-url=http://localhost:4566

🔟 Criar a API no API Gateway
aws apigateway create-rest-api --name "NotasFiscaisAPI" --endpoint-url=http://localhost:4566


A resposta será semelhante a:

{
  "id": "abc123",
  "name": "NotasFiscaisAPI"
}

11️⃣ Obter o ID do recurso raiz
aws apigateway get-resources --rest-api-id u0sk7fep5o --endpoint-url=http://localhost:4566


Saída esperada:

{
  "items": [
    {
      "id": "xyz456",
      "path": "/"
    }
  ]
}

12️⃣ Criar o recurso /notas na API
aws apigateway create-resource --rest-api-id u0sk7fep5o --parent-id onkhdnhrhl --path-part "notas" --endpoint-url=http://localhost:4566


Exemplo de resposta:

{
  "id": "mno789",
  "path": "/notas"
}

13️⃣ Configurar os métodos HTTP (POST e GET)
aws apigateway put-method --rest-api-id u0sk7fep5o --resource-id mhmc5ukc8z --http-method POST --authorization-type "NONE" --endpoint-url=http://localhost:4566
aws apigateway put-method --rest-api-id u0sk7fep5o --resource-id mhmc5ukc8z --http-method GET --authorization-type "NONE" --endpoint-url=http://localhost:4566

14️⃣ Integrar o método com a Lambda
aws apigateway put-integration --rest-api-id u0sk7fep5o --resource-id mhmc5ukc8z --http-method POST --type AWS_PROXY --integration-http-method POST --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:ProcessarNotasFiscais/invocations" --endpoint-url=http://localhost:4566

aws apigateway put-integration --rest-api-id u0sk7fep5o --resource-id mhmc5ukc8z --http-method GET --type AWS_PROXY --integration-http-method POST --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:ProcessarNotasFiscais/invocations" --endpoint-url=http://localhost:4566

15️⃣ Conceder permissão à API para invocar a Lambda
aws lambda add-permission --function-name ProcessarNotasFiscais --statement-id apigateway-access --action "lambda:InvokeFunction" --principal apigateway.amazonaws.com --source-arn "arn:aws:execute-api:us-east-1:000000000000:u0sk7fep5o/*/POST/notas" --endpoint-url=http://localhost:4566

16️⃣ Implementar a API
aws apigateway create-deployment --rest-api-id u0sk7fep5o --stage-name dev --endpoint-url=http://localhost:4566

17️⃣ Testar a API

No PowerShell:

Invoke-RestMethod -Uri "http://localhost:4566/restapis/u0sk7fep5o/dev/_user_request_/notas" `
  -Method POST `
  -ContentType "application/json" `
  -Body '{"id": "NF-999", "cliente": "João2 Silva", "valor": 1000.0, "data_emissao": "2025-01-31"}'


Ou com Python:

aws apigateway get-integration --rest-api-id u0sk7fep5o --resource-id mhmc5ukc8z --http-method POST --endpoint-url=

✅ Resultado Esperado

Arquivos enviados ao bucket S3 são processados automaticamente pela Lambda

Dados são armazenados no DynamoDB

Arquivos processados são movidos para a pasta sucesso

Todo o processo ocorre localmente via LocalStack
