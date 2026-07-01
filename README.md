# Desafio DIO - AWS Lambda + S3

Este projeto faz parte do desafio da **Digital Innovation One (DIO)** sobre tarefas automatizadas com **AWS Lambda** e **Amazon S3**.  
O objetivo é implementar uma função Lambda que processa arquivos enviados ao S3 e grava os dados no DynamoDB.

---

## Objetivo do Projeto
- Criar uma Lambda Function integrada ao S3.  
- Automatizar o processamento de arquivos JSON.  
- Persistir os dados em uma tabela DynamoDB.  

---

## Arquitetura da Solução

```text
[S3 Bucket: notas-fiscais-upload]
        ↓ Trigger
[Lambda: ProcessarNotasFiscais]
        ↓
[DynamoDB: Tabela NotasFiscais]
```

---

## Função Lambda

```text
aws lambda create-function --function-name ProcessarNotasFiscais \
    --runtime python3.9 \
    --role arn:aws:iam::000000000000:role/lambda-role \
    --handler grava_db.lambda_handler \
    --zip-file fileb://lambda_function.zip \
    --endpoint-url=http://localhost:4566
```

---

## Bucket S3

```text
awslocal s3api create-bucket --bucket notas-fiscais-upload
```

---

## Dynamo DB

```text
aws dynamodb create-table \
    --endpoint-url=http://localhost:4566 \
    --table-name NotasFiscais \
    --attribute-definitions AttributeName=id,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

---

## Configuração trigger do S3

```text
aws lambda add-permission --function-name ProcessarNotasFiscais \
    --statement-id s3-trigger-permission \
    --action "lambda:InvokeFunction" \
    --principal s3.amazonaws.com \
    --source-arn "arn:aws:s3:::notas-fiscais-upload" \
    --endpoint-url=http://localhost:4566

aws s3api put-bucket-notification-configuration \
    --bucket notas-fiscais-upload \
    --notification-configuration file://notification_roles.json \
    --endpoint-url=http://localhost:4566
```

---

## Testar envio de arquivo

```text
aws s3 cp notas_fiscais_2025.json s3://notas-fiscais-upload --endpoint-url=http://localhost:4566
```
