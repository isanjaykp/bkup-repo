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
                sh '''
              echo 'Install kubectl'
              curl -LO "https://storage.googleapis.com/kubernetes-release/release/\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl
function get_jenkins_pod_id {
                kubectl get pods -n jenkins -l app.kubernetes.io/component=jenkins-master -o custom-columns=PodName:.metadata.name | grep jenkins-
              }
  
              echo 'Create jenkins backup'
              kubectl exec $(get_jenkins_pod_id) -- bash -c 'cd /var; \
                rm -rf jenkins_backup; \
                mkdir -p jenkins_backup; \ 
                cp -r jenkins_home jenkins_backup/jenkins_home; \
                tar -zcvf jenkins_backup/jenkins_backup.tar.gz jenkins_backup/jenkins_home'
              
              cd && kubectl cp jenkins/$(get_jenkins_pod_id):/var/jenkins_backup/jenkins_backup.tar.gz jenkins_backup.tar.gz
              
              echo 'Upload jenkins_backup.tar to S3 bucket'
              aws s3 cp jenkins_backup.tar.gz s3://jenkins-backups/$(date +%Y%m%d%H%M)/jenkins_backup.tar.gz
              
              echo 'Remove files after succesful upload to S3'
              kubectl exec $(get_jenkins_pod_id) -- bash -c 'rm -rf /var/jenkins_backup'
            '''
              }
          
        }
      }
    }
  }
}
