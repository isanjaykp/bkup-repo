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
    choice(
      name: 'REGION',
      choices: [
          ' ',
          'us-east-1',
          'us-east-2'
          ],
      description: 'AWS Account Region'
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
                sh 'aws s3 cp s3://dx-cf-bucket/application.yml .'
                sh 'aws s3 cp README.md s3://dx-cf-bucket/README.md'
                sh 'ls -ltr'
                sh 'echo "touch .."'
                sh 'touch jenkins_backup.txt'
                sh 'echo "cp .."'
                sh 'sudo aws s3 cp jenkins_backup.txt s3://jdx-devops-backup/$(date +%Y%m%d%H%M)_jenkins_backup.txt'
                sh 'echo "done .."'
                sh '''
              echo 'Install kubectl'
              curl -LO "https://storage.googleapis.com/kubernetes-release/release/\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl

              echo 'Create jenkins backup'
              
              touch jenkins_backup.txt
              echo 'Upload jenkins_backup.txt to S3 bucket'
              aws s3 cp jenkins_backup.txt s3://jdx-devops-backup/$(date +%Y%m%d%H%M)/jenkins_backup.txt
           
              echo 'Remove files after succesful upload to S3'
              '''
              }
          
        }
      }
    }
  }
}
