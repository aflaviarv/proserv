=============================
Bot de CloudWatch no Telegram
=============================

Objetivo
--------
Na intenção de dinamizar a disponibilidade de acesso a informações importantes de logs que são fornecidas pelo AWS CloudWatch, esse bot traz as infos por meio do canal de comunicação Telegram.

Passo a Passo
-------------
1 - Criar o Token do BOT do telegram:

	Seguir os passos do site oficial do telegram:
	
		https://core.telegram.org/bots/features#botfather
		

	Com o Token Criado, criar um grupo ou caso queira que o bot retorne as mensagens para o seu número de celular, mande uma mensagem para o bot ou no grupo.

	Essa mensagem vai gerar um ID e esse ID você vai pegar usando a url abaixo no navegador:
	
	https://api.telegram.org/bot123456789789:TOKEN-DO-BOT/GetUpdates
	
	
	``Response:	"message":{"message_id":62,"from":{"id":198XXXXXX,"is_bot":false,``

	Vai puxar algo parecido com isso e você tem que pegar o ID do ``"from":{"id":19817XXXXX,``
	
	``Exemplo de ID -> 19817XXXXX``

Esse é o ID que vamos usar no código na linha: 41, é o ID do grupo ou chat que o BOT vai enviar a mensagem.
	

2 - Criar um "IAM role" para usarmos no lambda com as seguintes permissões:

	A - Crie uma policy com o json:
	
``Exemplo -> 	{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "sns:Publish",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:*:*:*",
                "arn:aws:sns:*:*:*"
            ]
        }
    ]
}``
	
	B - Criar IAM role como serviço para o Lambda e colocar a politica nova.
	

3 - SNS criar um Topic com o nome que desejar. (Standard)

		
4 - Lambda(Author from scratch).
	
	A - Crie a função com o nome que desejar; 
	B - Use python 3.9;
	C - Associar o IAM Role que criamos;
	D - Dentro da função você vai fazer o upload do telegram-alarm.zip
	E - Vá em Configuration e crie uma trigger apontando para o SNS que criou.
	F - Em configuration do lambda coloque uma variavel de ambiente como:
		
		``Key: TELEGRAM_CHAT_ID | Value: o número do chat Id que pegamos no começo.``
		
5 - Secrets Manager:
	
	A - Store a new secret (Guardar novo segredo) 
	B - Selecione o tipo como: Outro tipo de segredo (API key, OAuth token, other.):
	
		Coloque: 
			``Key/Value:TELEGRAM_BOT_TOKEN | Plaintext: TOKEN DO SEU BOT!``
	
6 - Agora é só testar o código usando um json de alarme:

	``Exemplo -> 	{
    "AlarmName": "teste-telegram",
    "AlarmDescription": null,
    "AWSAccountId": "99999999999999",
    "AlarmConfigurationUpdatedTimestamp": "2023-03-06T17:06:31.446+0000",
    "NewStateValue": "ALARM",
    "NewStateReason": "Threshold Crossed: 1 out of the last 1 datapoints [0.33574153327928674 (06/03/23 17:03:00)] was greater than the threshold (0.0) (minimum 1 datapoint for OK -> ALARM transition).",
    "StateChangeTime": "2023-03-06T17:09:40.215+0000",
    "Region": "US East (Ohio)",
    "AlarmArn": "arn:aws:cloudwatch:us-east-2:99999999999999:alarm:teste-telegram",
    "OldStateValue": "OK",
    "AlarmDescription": "Texto de exemplo de descrição",
    "OKActions": [],
    "AlarmActions": [
        "arn:aws:sns:us-east-2:99999999999999:sns-telegram-teste"
    ],
    "InsufficientDataActions": [],
    "Trigger": {
        "MetricName": "CPUUtilization",
        "Namespace": "AWS/EC2",
        "StatisticType": "Statistic",
        "Statistic": "AVERAGE",
        "Unit": null,
        "Dimensions": [
            {
                "value": "i-0b670cc48c15a9514",
                "name": "InstanceId"
            }
        ],
        "Period": 60,
        "EvaluationPeriods": 1,
        "DatapointsToAlarm": 1,
        "ComparisonOperator": "GreaterThanThreshold",
        "Threshold": 0.0,
        "TreatMissingData": "missing",
        "EvaluateLowSampleCountPercentile": ""
    }
}``

7 - Para funcionar precisa que no seu Alarme tenha o Actions apontado para o SNS que ativa o lambda!

Dessa forma, você pode ter as seguintes notificações em seu Telegram.

.. images:: /images/primeiraformachat.png

.. images:: /images/segundaformachat.png    

.. images:: /images/terceiraformachat.png    