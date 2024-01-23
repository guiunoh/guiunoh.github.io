---
title: "amplify backend"
categories:
  - "development"
tags:
  - "aws"
  - "amplify"
date: "2021-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---

### 모델 자기 참조
* @model은 dynamodb 를 사용하여 자동으로 resolver를 생성
* @key를 사용하면 GSI 생성
* @connection 사용하여 GSI와 필드 연결
* 연결을 위한 별도 ID 필드 필요
  ```graphql
  type LinkedListNode @model @key(name: "ByNext", fields: ["nextId"]) {
      id: ID!
      data: String!
      nextId: ID!
      next: LinkedListNode @connection(fields: ["nextId"])
  }

  type TreeNode @model @key(name: "ByParent", fields: ["parentID"]) {
      id: ID!
      data: String!
      parentID: ID!
      parent: TreeNode @connection(fields: ["parentID"])
      childrens: [TreeNode] @connection(keyName: "ByParent", fields: ["id"])
  }
  ```


### custom resolver
  * @model에 의해 생성 된 DynamoDB 테이블에 대해보다 구체적인 쿼리를 작성이 필요한 경우
  * GSI를 쿼리하는 최상위 쿼리 필드를 생성하지 않으므로 이에 대한 최상위 쿼리가 필요한 경우
  * 아래의 예에서 Comment에 생성되는 gsi-TodoComments 테이블에 대한 최상위 resolver는 생성 되지 않는다.
    ```graphql
    type Todo @model {
      id: ID!
      name: String!
      description: String
      comments: [Comment] @connection(name: "TodoComments")
    }
    type Comment @model {
      id: ID!
      content: String
      todo: Todo @connection(name: "TodoComments")
    }
    ```
* gsi-TodoComments resolver 생성 순서
* 다음과 같이 schema.graphql에 추가
  ```graphql
  type CommentConnection {
    items: [Comment]
    nextToken: String
  }
  type Query {
    commentsForTodo(todoId: ID!, limit: Int, nextToken: String): CommentConnection
  }
  ```
* amplify/backend/api/chatpuleangql/stacks/ 폴더 아래에 있는 CustomResource.json 파일을 별도의 이름으로 하나 생성( CommentConnection.CustomResource.json )
* DataSource와 리졸버 템플릿 연결 내용
* EmptyResource 의 내용을 아래와 같이 수정
  ```json
  "QueryCommentsForTodoResolver": {
    "Type": "AWS::AppSync::Resolver",
    "Properties": {
      "ApiId": {
        "Ref": "AppSyncApiId"
      },
      "DataSourceName": "CommentTable",
      "TypeName": "Query",
      "FieldName": "commentsForTodo",
      "RequestMappingTemplateS3Location": {
        "Fn::Sub": [
          "s3://${S3DeploymentBucket}/${S3DeploymentRootKey}/resolvers/Query.commentsForTodo.req.vtl",
          {
            "S3DeploymentBucket": {
              "Ref": "S3DeploymentBucket"
            },
            "S3DeploymentRootKey": {
              "Ref": "S3DeploymentRootKey"
            }
          }
        ]
      },
      "ResponseMappingTemplateS3Location": {
        "Fn::Sub": [
          "s3://${S3DeploymentBucket}/${S3DeploymentRootKey}/resolvers/Query.commentsForTodo.res.vtl",
          {
            "S3DeploymentBucket": {
              "Ref": "S3DeploymentBucket"
            },
            "S3DeploymentRootKey": {
              "Ref": "S3DeploymentRootKey"
            }
          }
        ]
      }
    }
  }
  ```
* amplify/backend/api/chatpuleangql/resolvers/ 폴더 아래에 리졸버 템플릿을 생성(request, response 각각 생성)
* 이름 규칙에 따라서 Query.commentsForTodo.req.vtl, Query.commentsForTodo.res.vtl 으로 생성
  ```velocity
  ## Query.commentsForTodo.req.vtl **
  #set( $limit = $util.defaultIfNull($context.args.limit, 10) )
  {
    "version": "2017-02-28",
    "operation": "Query",
    "query": {
      "expression": "#connectionAttribute = :connectionAttribute",
      "expressionNames": {
          "#connectionAttribute": "commentTodoId"
      },
      "expressionValues": {
          ":connectionAttribute": {
              "S": "$context.args.todoId"
          }
      }
    },
    "scanIndexForward": true,
    "limit": $limit,
    "nextToken": #if( $context.args.nextToken ) "$context.args.nextToken" #else null #end,
    "index": "gsi-TodoComments"
  }
  ```
  ```velocity
  ## Query.commentsForTodo.res.vtl **
  $util.toJson($ctx.result)
  ```


### lambda resolver
* @function 지시문을 사용하면 람다 해석기를 AppSync API에 연결할 수 있다.
* 다음과 같은 사용자 생성 Query를 람다와 연결 하는 방법
  * 예) echo Query
    * echofunction 의 이름을 가지고 있는 함수와 연결
    * 배포 스테이지에 따라 람다 함수 이름은 "-${env}"이 자동으로 붙여서 생성된다.
      ```graphql
      type Query {
        echo(msg: String): String @function(name: "echo-${env}")
      }
      ```
* amplify add function 으로 함수 추가한다. 이름은 위에 정의에 따라 "echo"로 한다.
  * 함수 생성 설정은 ‘Hello World’로 하고 다른 설정은 default를 따른다.(참고로 문서에는 express.js 로 생성해도 된다고 되어 있음)
* amplify mock를 실행하여 Query에 echo가 추가 되었음을 확인 가능


### lambda pipeline resolver
   * @function을 2개 이상 정의하여 pipeline resolveer를 생성
   * amplify add function으로 함수를 각각 추가(worker, audit)
   * 정의 순서대로 실행
    ```graphql
    type Mutation {
      doSomeWork(msg: String): String @function(name: "worker-${env}") @function(name: "audit-${env}")
    }
    ```


### add custom aws resource
* 참고
  * [Amplify Framework Documentation](https://docs.amplify.aws/cli/usage/customcf)
  * [How to add an SQS queue to your Amplify CLI bootstrapped project](https://medium.com/@navvabian/how-to-add-an-sqs-queue-to-your-amplify-cli-bootstrapped-project-cb7781c636ed)
* amplify/backend/backend-config.json 수정
  ```json
  "queue": { // custome category name
    "eventmessage": { // custeom resource name
      "service": "SQS",
      "providerPlugin": "awscloudformation"
    }
  }
  ```
* amplify/backend 에 폴더 구성 추가
  * "YOUR_QUEUE_NAME-template.json"& "YOUR_QUEUE_NAME-parameter.json" 패턴을 권고
    ```bash
    amplify
      \backend
        \queue
          \eventmessage
            parameters.json
            template.json
    ```
* template.json 내용
  ```json
  {
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "Queue resource stack creation using Amplify CLI",
    "Parameters": {
      "env": {
         "Type": "String"
      }
    },
    "Conditions": {
      "ShouldNotCreateEnvResources": {
        "Fn::Equals": [
          {
            "Ref": "env"
          },
          "NONE"
        ]
      }
    },
    "Resources": {
      "SQS": {
        "Type": "AWS::SQS::Queue",
        "Properties": {
          "QueueName": {
            "Fn::If": [
              "ShouldNotCreateEnvResources",
              "eventmessage", // custeom resource name
              {
                "Fn::Join": [
                  "",
                  [
                    "eventmessage", // custeom resource name
                    "-",
                    {
                      "Ref": "env"
                    }
                  ]
                ]
              }
            ]
          }
        }
      }
    },
    "Outputs": {
      "Name": {
        "Value": {
          "Ref": "SQS"
        }
      },
      "Arn": {
        "Value": {
          "Fn::GetAtt": [
            "SQS",
            "Arn"
          ]
        }
      },
      "Region": {
        "Value": {
          "Ref": "AWS::Region"
        }
      }
    }
  }
  ```
* amplify add function를 사용하여 consumer 함수 추가
* backend-config.json을 수정하여 큐 종속성 추가
  ```json
  "eventmessageconsumer": { // lambda function name
    "build": true,
    "providerPlugin": "awscloudformation",
    "service": "Lambda",
    "dependsOn": [
      {
        "category": "queue", // custeom category name
        "resourceName": "eventmessage", // custeom resource name
        "attributes": [
          "Name",
          "Arn"
        ]
      }
    ]
  ```

* amplify cli 자동 생성 구성을 변경하는 경우 변경후 반드시 "amplify env checkout YOUR_ENVIRONMENT" 를 실행한다.
* consumer 함수 template 수정
* amplify/backend/function/{consumer-function-name}/template.json
* Parameters 추가
  ```json
  "Parameters": {
  .....
    "queueeventmessageName": {
        "Type": "String",
        "Default": "queueeventmessageName"
      },
      "queueeventmessageArn": {
        "Type": "String",
        "Default": "queueeventmessageArn"
      }
    }
  }
  ```
* Environment 추가
  ```json
  "Resources": {
  .....
    "LambdaFunction": {
        "Environment": {
          "Variables": {
            "EVENT_MESSAGE_QUEUE": {
              "Ref": "queueeventmessageName"
            }
          }
        }
      }
    },
  .....
  }
  ```
* lambdaexecutionpolic 추가
  ```json
  "Resources": {
    "lambdaexecutionpolicy": {
      "DependsOn": [
        "LambdaExecutionRole"
      ],
      "Type": "AWS::IAM::Policy",
      "Properties": {
        "PolicyName": "lambda-execution-policy",
        "Roles": [
          {
            "Ref": "LambdaExecutionRole"
          }
        ],
        "PolicyDocument": {
          "Version": "2012-10-17",
          "Statement": [
            ....
            {
              "Effect": "Allow",
              "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
              ],
              "Resource": "*"
            }
          ]
        }
      }
    }
  },
  ```
* 이벤트 소스 맵핑 (SQS & lambda)
  ```json
  "Resources": {
    "LambdaFunctionEventSourceMapping": {
      "DependsOn": [
        "lambdaexecutionpolicy"
      ],
      "Type": "AWS::Lambda::EventSourceMapping",
      "Properties": {
        "EventSourceArn": {
          "Ref": "queueeventmessageArn"
        },
        "FunctionName": {
          "Ref": "LambdaFunction"
        }
      }
    },
  }
  ```
* 변경사항 반영 cli 명령
  * amplify env checkout YOUR_ENVIRONMENT
  * amplify push — yes


### Dynamodb 모드 설정
* appsync api 이름 아래에 parameters.json 를 변경하여 Dynamodb 빌링 모드를 수정 할수 있다.```
  ```json
  {
    "DynamoDBBillingMode": "PAY_PER_REQUEST", // PROVISIONED
    "DynamoDBModelTableReadIOPS": 5,
    "DynamoDBModelTableWriteIOPS": 5
  }
  ```

### Dynamodb TTL 설정
* [flogy/graphql-ttl-transformer](https://github.com/flogy/graphql-ttl-transformer)
  * 공식 문서에도 언급이 되어 있음
