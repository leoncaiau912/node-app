pipeline {
  agent {
    docker {
      image 'leoncaiau912/build-container:latest'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
    stage('Create Packer AMI') {
        steps {
          withCredentials([
            usernamePassword(credentialsId: '26be8d06-879b-4aec-b68a-e5fcb4f3ab26', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
          ]) {
            sh 'packer build -var aws_access_key=${AWS_KEY} -var aws_secret_key=${AWS_SECRET} packer/packer.json'
        }
      }
    }
    stage('AWS Deployment') {
      steps {
          withCredentials([
            usernamePassword(credentialsId: '26be8d06-879b-4aec-b68a-e5fcb4f3ab26', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
            usernamePassword(credentialsId: 'dd50d755-8d40-4782-808e-17c016f0bc67', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER'),
          ]) {
            sh 'rm -rf node-app-terraform'
            sh 'git clone https://github.com/leoncaiau912/node-app-terraform.git'
            sh '''
               cd node-app-terraform
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="leon cai" -c user.email="leoncaiau912@gmail.com" commit -m "terraform state update from Jenkins"
               git push https://${REPO_USER}:${REPO_PASS}@github.com/leoncaiau912/node-app-terraform.git master
            '''
        }
      }
    }
  }
  environment {
    npm_config_cache = 'npm-cache'
  }
}
