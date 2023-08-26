pipeline {
  agent {
    kubernetes {
      label 'jenkins-backup-job'
      defaultContainer 'jnlp'
      yamlFile 'k8s/build-pod.yaml'
    }
  }
parameters {
    string(name: 'STACK_NAME', defaultValue: 's3-stack', description: 'Enter the CloudFormation Stack Name.')
    string(name: 'PARAMETERS_FILE_NAME', defaultValue: 's3-stack-parameters.properties', description: 'Enter the Parameters File Name (Must contain file extension type *.properties)')
    string(name: 'TEMPLATE_NAME', defaultValue: 'S3-Bucket.yaml', description: 'Enter the CloudFormation Template Name (Must contain file extension type *.yaml)')
    credentials(name: 'CFN_CREDENTIALS_ID', defaultValue: '', description: 'AWS Account Role.', required: true)
    choice(
      name: 'REGION',
      choices: [
          ' ',
          'us-east-1',
          'us-east-2'
          ],
      description: 'AWS Account Region'
    )
    choice(
      name: 'ACTION',
      choices: ['create-changeset', 'execute-changeset', 'deploy-stack', 'delete-stack'],
      description: 'CloudFormation Actions'
    )
    booleanParam(name: 'TOGGLE', defaultValue: false, description: 'Are you sure you want to perform this action?')
  }
  options {
    buildDiscarder(logRotator(numToKeepStr:'30'))
    timeout(time: 60, unit: 'MINUTES')
  }
stages {
    stage('Backup Jenkins'){
      steps {
        container('awscli'){
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: "${CFN_CREDENTIALS_ID}",
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                sh 'aws --version'
                sh 'aws s3 ls'
              }
          
        }
      }
    }
  }
}
