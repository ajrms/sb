pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }

  environment {
    gitName = 'ajrms'
    gitEmail = 'sun98415@gmail.com'
    gitWebaddress = 'https://github.com/ajrms/sb.git'
    gitSshaddress = 'git@github.com:ajrms/sb.git'
    gitCredential = 'git_cre' // github credential 생성 시의 ID
    dockerHubRegistry = 'ajrms/sb'
    dockerHubRegistryCredential = 'docker_cre'
  }

  stages {
    stage('checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
        }
        post {
            failure {
                echo 'Repository clone failure'
            }
            success {
                echo 'Repository clone success'
            }
        }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함
        }
        post {
            failure {
                echo 'Maven build failure'
            }
            success {
                echo 'Maven build success'
            }
        }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        // ajrms/sb:4 이런식으로 빌드가 될 것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수
        }
        post {
            failure {
                echo 'docker image build failure'
            }
            success {
                echo 'docker image build success'
            }
        }
    }
    stage('docker image push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
          }
        }
        post {
            failure {
                echo 'docker image push failure'
                sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry}:latest"
            }
            success {
                sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry}:latest"
                echo 'docker image push success'
            }
        }
    }
    stage('docker container deploy') {
      steps {
        sh 'docker rm -f sb'
        sh "docker run -dp 5656:8085 --name sb ${dockerHubRegistry}:${currentBuild.number}"
        }
        post {
            failure {
                echo 'docker container deploy failure'
            }
            success {
                echo 'docker container deploy success'
            }
        }
    }
  }
}
