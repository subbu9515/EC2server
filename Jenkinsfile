//@Library(['release@master', 'sharedlibrary@release/3.0.0'])
env.ARN = getAssumedArn("")
env.TEMPLATE_FILE  = 'server.yaml'
env.STACK_NAME = 'Linuxserver'
env.AWS_REGION = 'us-east-1'
env.SESSION_NAME = 'jenkins'

pipeline {
agent any

  stages {
    stage('git Checkout') {
      steps {
        // where branch?
        //git url:https://tfs-glo-lexisadvance.visualstudio.com/DefaultCollection/lncanada/_git/Windows%20Server%20Upgrade, branch: <branch_name>
        git branch: 'main', changelog: false, poll: false, url: 'https://github.com/subbu9515/EC2server.git'
      }
    }

    stage('Assume Role') {
      steps {
        script {
          // why region?
          def credsJson = sh( script: "aws sts assume-role --role-arn '${env.ARN}' --role-session-name '${env.SESSION_NAME}' --region ${env.AWS_REGION}", returnStdout: true).trim()
          def creds = readJSON(credsJson)
          env.AWS_ACCESS_KEY_ID = creds.Credentials.AccessKeyId
          env.AWS_SECRET_ACCESS_KEY = creds.Credentials.SecretAccessKey
          env.AWS_SESSION_TOKEN = creds.Credentials.SessionToken
        }
      }
    }
    
    stage("Validate CFN Template") {
      steps {
        withEnv([
          "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
          "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
          "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
        ])
        {
          sh "aws cloudformation validate-template --template-body file://${env.TEMPLATE_FILE} --region ${env.AWS_REGION}"
        }
      }
    }

    stage('CFN Stack Create') {
      steps {
        withEnv([
          "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
          "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
          "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
        ]) {
          sh """
            aws cloudformation deploy \
              --stack-name ${env.STACK_NAME} \
              --template-file ${env.TEMPLATE_FILE} \
              --region ${env.AWS_REGION} \
              --capabilities CAPABILITY_NAMED_IAM \
              --parameter-overrides \\
                  AmiID=ami-0013b6db63dc8ec39 \\
                  InstanceType=t2.micro \\
                  VpcId=vpc-01fe21cb82763266a \\
                  SubnetId=subnet-0fbe8b22d99cc2a8d
            """

        }
      }
    }

    stage('Outputs') {
      steps {
        sh """aws cloudformation describe-stacks --stack-name ${env.STACK_NAME} --region ${env.AWS_REGION} --query "Stacks[0].Outputs""""
      }
    }
  }
}


def getAssumedArn(Environment){
    return 'arn:aws:iam::221082192077:role/Admin'

  }


def readJSON(String text) {
    def accessKeyId = sh(script: "echo '${text}' | jq -r '.Credentials.AccessKeyId'", returnStdout: true).trim()
    def secretAccessKey = sh(script: "echo '${text}' | jq -r '.Credentials.SecretAccessKey'", returnStdout: true).trim()
    def sessionToken = sh(script: "echo '${text}' | jq -r '.Credentials.SessionToken'", returnStdout: true).trim()

    return [
        Credentials: [
            AccessKeyId: accessKeyId,
            SecretAccessKey: secretAccessKey,
            SessionToken: sessionToken
        ]
    ]
}
