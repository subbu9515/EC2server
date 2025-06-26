// changes: uncomment this line
//@Library(['release@master', 'sharedlibrary@release/3.0.0'])

// changes: env.Environment
env.ARN = getAssumedArn(env.Environment)

// changes: change template file location
env.TEMPLATE_FILE  = 'server.yaml'
env.STACK_NAME = 'Linuxserver'
env.AWS_REGION = 'ca-central-1'
env.SESSION_NAME = 'jenkins-linux-server'

pipeline {
  agent any
//changes: need this agent
//agent {
//  node {
//    label 'DockerAgent'
//  }
//}

  stages {
    stage('git Checkout') {
      steps {
        // where branch?
        //git url:https://tfs-glo-lexisadvance.visualstudio.com/DefaultCollection/lncanada/_git/Windows%20Server%20Upgrade, branch: <branch_name>
        // changes: change url and branch
        git branch: 'main', changelog: false, poll: false, url: 'https://github.com/subbu9515/EC2server.git'
      }
    }

    stage('Assume Role') {
      steps {
        script {
          def credsJson = sh( script: "aws sts assume-role --role-arn '${env.ARN}' --role-session-name '${env.SESSION_NAME}' --region ${env.AWS_REGION}", returnStdout: true).trim()
          // use readJSON from shared library
          // def creds = readJSON text: credsJson
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
        sh """
          aws cloudformation describe-stacks --stack-name ${env.STACK_NAME} --region ${env.AWS_REGION} --query 'Stacks[0].Outputs'
          """
      }
    }
  }
}


def getAssumedArn(Environment){
    // uncomment this
    //switch(Environment) {
    //  case 'Dev': return 'arn:aws:iam::393012300164:role/AssetApplication_1939/JenkinsExecutionRole'
     // default: return 'arn:aws:iam::393012300164:role/AssetApplication_1939/JenkinsExecutionRole'
    //}
    return 'arn:aws:iam::221082192077:role/Admin'
}


// changes: remove this func
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
