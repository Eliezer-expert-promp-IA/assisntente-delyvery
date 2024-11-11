# assisntente-delyvery
Criação de do assistente de delivere pelo Amazom Step Functions e Bedrock

# Essa ativida ensiou a utilizar os serviços da AMAZOM Step Fuctions e Bedrock
- Set Fuctions
  * Usado para efetuar gerenciamento e orquestar multiplas aplicações que são isoladas
 
*  Bedrock
  * Gerenciado de tecnologias LLM - IAs, onde pode ser acessadas e ter liberação de acordo com a região que sua conta estiver configurada.

# PROJETO ESTA EM ASL - JSON, LINGUA QUE A AMAZONSETP FUCTIONS USA POR PADRÃO
A Maquina estado criado foi:

# ilustação:
![image](https://github.com/user-attachments/assets/c60a337c-1a00-4a77-bb9b-13f1a9622bd3)


#Código:#

{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Pergunta de comida",
  "States": {
    "Pergunta de comida": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "us.meta.llama3-2-1b-instruct-v1:0",
        "Body": {
          "inputText": "Uma sugestão de comida para acompanhar um amigo",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add first result to conversation history",
      "ResultPath": "$.result_one",
      "ResultSelector": {
        "result_one.$": "$.Body.generations[0].text"
      }
    },
    "Add first result to conversation history": {
      "Type": "Pass",
      "Next": "Pergunta de bebida",
      "Parameters": {
        "convo_one.$": "States.Format('{}\n{}', $.prompt_one, $.result_one.result_one)"
      },
      "ResultPath": "$.convo_one"
    },
    "Pergunta de bebida": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "us.meta.llama3-2-1b-instruct-v1:0",
        "Body": {
          "prompt.$": "States.Format('{}\n{}', $.convo_one.convo_one, $.prompt_two)",
          "max_tokens": 200
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add second result to conversation history",
      "ResultSelector": {
        "result_two.$": "$.Body.generations[0].text"
      },
      "ResultPath": "$.result_two"
    },
    "Add second result to conversation history": {
      "Type": "Pass",
      "Next": "pergunta de local",
      "Parameters": {
        "convo_two.$": "States.Format('{}\n{}\n{}', $.convo_one.convo_one, $.prompt_two, $.result_two.result_two)"
      },
      "ResultPath": "$.convo_two"
    },
    "pergunta de local": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "us.meta.llama3-2-1b-instruct-v1:0",
        "Body": {
          "prompt.$": "States.Format('{}\n{}', $.convo_two.convo_two, $.prompt_three)",
          "max_tokens": 250
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "End": true,
      "ResultSelector": {
        "result_three.$": "$.Body.generations[0].text"
      }
    }
  }
}
