# -Criando-um-Assistente-de-Delivery-com-AWS-Step-Functions-e-Bedrock
import boto3

# Crie uma máquina de estado no Step Functions
step_functions = boto3.client('stepfunctions')
state_machine_arn = 'arn:aws:states:us-east-1:123456789012:stateMachine:my-state-machine'

# Configure os parâmetros do modelo
model_id = 'cohere.command-text-v14'
prompt = 'Qual é o melhor restaurante em São Paulo?'
max_tokens = 250

# Invoque o Bedrock para executar inferências em um modelo de linguagem natural
bedrock = boto3.client('bedrock')
response = bedrock.invoke_model(
    ModelId=model_id,
    Body={'prompt': prompt, 'max_tokens': max_tokens},
    ContentType='application/json',
    Accept='*/*'
)

# Use o resultado da inferência para determinar o próximo passo no workflow
result = response['Body']['generations'][0]['text']
if result == 'Sim':
    # Crie uma função do AWS Lambda que unifique as respostas das tarefas paralelas em uma única resposta e gere um resultado
    lambda_function = boto3.client('lambda')
    response = lambda_function.invoke(
        FunctionName='my-lambda-function',
        InvocationType='RequestResponse',
        Payload={'result': result}
    )
    print(response['Payload']['result'])
else:
    print('Não')
