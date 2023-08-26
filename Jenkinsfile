pipeline {
  agent {
    kubernetes {
      label 'jenkins-backup-job'
      defaultContainer 'jnlp'
      yamlFile 'k8s/build-pod.yaml'
    }
  }
  parameters {
   
    credentials(name: 'CFN_CREDENTIALS_ID', defaultValue: '', description: 'AWS Account Role.', required: true)
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
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                sh 'sudo aws s3 ls'
          }
        }
      }
    }
  }
}
