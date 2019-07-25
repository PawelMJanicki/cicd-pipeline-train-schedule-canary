pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "pawelmjanicki/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
          when {
            branch 'master'
          }
          environment {
            CANARY_REPLICAS = 2
          }
          steps{
            kubernetesDeploy(
              kubeconfigId: 'kubeconfig', //value stored as a kubernetes config credential on the Jenkins (names must much)
              configs: 'train-schedule-kube-canary.yml', //Kubernetes deploy document containing application setup
              enableConfigSubstitution: true //Jenkins plugin will make substitutions for $ signed variables
            )
          }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment {
              CANARY_REPLICAS = 0 //Cleaning Canary pods that are no longer needed
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                  kubeconfigId: 'kubeconfig', //value stored as a kubernetes config credential on the Jenkins (names must much)
                  configs: 'train-schedule-kube-canary.yml', //Kubernetes deploy document containing application setup
                  enableConfigSubstitution: true //Jenkins plugin will make substitutions for $ signed variables
                )
                kubernetesDeploy(
                  kubeconfigId: 'kubeconfig', //value stored as a kubernetes config credential on the Jenkins (names must much)
                  configs: 'train-schedule-kube.yml', //Kubernetes deploy document containing application setup
                  enableConfigSubstitution: true //Jenkins plugin will make substitutions for $ signed variables
                )
            }
        }
    }
}
