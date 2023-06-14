pipeline {
  agent any

  stages {
    stage('Clone repository') {
      steps {
        git branch: 'dev', url: 'https://github.com/yashwanthhvx/Python_App.git'
      }
    }
    
    stage('Show branch name') {
      steps {
        script {
          def branchName = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
          echo "Executing from branch: ${branchName}"
        }
      }
    }
    
    stage('Copy the code packages to destination path') {
      steps {
        sh 'sudo cp -R /var/lib/jenkins/workspace/Jenkins_App_dev/python-app/* /var/lib/jenkins/workspace/Python_App_dev/Python-App/.'
      }
    }
      
    stage('Build Docker image') {
      steps {
        dir('Python-App') {
          sh 'sudo docker build -t python-app:latest .'
        }
      }
    }

    stage('Run Docker container') {
      steps {
        dir('Python-App') {
          sh 'sudo docker run -p 8000:5000 -d python-app:latest'
        }
      }
    }

    stage('Execute the Bash Script to docker login and save image') {
      steps {
        dir('Python-App') {
          sh 'sudo bash docker-credentials.sh'
          sh 'sudo docker tag python-app:latest hvxuser/python-jenkins'
          sh 'sudo docker push hvxuser/python-jenkins'
        }
      }
    }

    stage('Create Kubernetes Deployment') {
      steps {
        dir('Python-App') {
          script {
            def deployCmd = "sudo kubectl apply -f k8-deployment.yml"
            def deployStatus = sh(script: deployCmd, returnStatus: true)

            if (deployStatus == 0) {
              echo "Kubernetes deployment succeeded"
              currentBuild.result = 'SUCCESS'
            } else {
              echo "Kubernetes deployment failed"
              currentBuild.result = 'FAILURE'
            }
          }
        }
      }
    }

    stage('Create Kubernetes Service') {
      steps {
        dir('Python-App') {
          script {
            def serviceCmd = "sudo kubectl apply -f k8-service.yml"
            def serviceStatus = sh(script: serviceCmd, returnStatus: true)

            if (serviceStatus == 0) {
              echo "Kubernetes service creation succeeded"
            } else {
              echo "Kubernetes service creation failed"
            }
          }
        }
      }
    }
  }
}
