pipeline {

  environment {
    dockerimagename = "oussamaw/nodeapp"
    dockerImage = ""
  }

  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/Oussama-Goumghar/nodeapp_test.git'
      }
    }
    
    // stage('SonarQube analysis') {
    //   steps {
    //     def scannerHome = tool 'sonarqube';
    //     withSonarQubeEnv('node-app') {
    //       sh "${scannerHome}/bin/sonar-scanner \
    //       -D sonar.login=admin \
    //       -D sonar.password=sonar \
    //       -D sonar.projectKey=node-app \
    //       -D sonar.host.url=http://10.108.157.136/"
    //     }  
    //   }  
    // }
  
    // stage('Quality Gates'){
    //   steps {
    //     timeout(time: 2, unit: 'MINUTES') {
    //       waitForQualityGate abortPipeline: true
    //     }
    //   }   
    // }

    stage('Build-Docker-Image') {
      steps {
        container('docker') {
          sh 'docker build -t oussamaw/nodeapp:latest .'
        }
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'docker_hub_login'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
        }
      }
    }

  }

}
