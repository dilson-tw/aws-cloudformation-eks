pipeline {
    environment {
        COMMIT_ID = ''
        REGISTRY = 'my-docker-registry'
    }
    agent any
    stages {
        stage ('checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'dilson-github', url: 'https://github.com/dilson-tw/aws-cloudformation-eks']]])
                }
            }
        }

        stage ('Build') {
            steps {
                script {
                    gitCommitHash = sh (script: 'git rev-parse --short HEAD', returnStdout: true)
                    withDockerRegistry([credentialsId: 'dilson-dockerhub', url: 'https://index.docker.io/v1/']) {
                        sh "docker build -f api/Dockerfile ./api -t sonild/basic_example:${gitCommitHash}"
                        sh "docker push sonild/basic_example:${gitCommitHash}"
                    }
                }
            }
        }
      stage ('K8S Deploy') {
          steps {
                script {
                    gitCommitHash = sh (script: 'git rev-parse --short HEAD', returnStdout: true)
                    COMMIT_ID = gitCommitHash
                    kubernetesDeploy(
                        configs: 'k8s/pipeline_deployment.yaml',
                        kubeconfigId: 'K8S',
                        enableConfigSubstitution: true
                    )
                    // withKubeConfig([credentialsId: 'K8S', serverUrl: 'https://58919550601C3EA7F21AAC0AC9B3ED75.gr7.us-west-2.eks.amazonaws.com']) {
                    //     sh 'kubectl apply -f k8s/pipeline_deployment.yaml'
                    //     }
                }
            }
      }
    }
}
