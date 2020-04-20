pipeline {
    agent any

    tools {
        maven 'mvn-v3.6.0'
        jdk 'jdk8'
    }
    
environment {
        DOCKER_CREDENTIALS = credentials('docker-registry-credentials')
        CF_CREDENTIALS = credentials('pcfcreds')
    }
    
    stages {
        stage('Build') {
            steps {
                // echo 'Printing the docker registry username and password'
                // sh "echo $DOCKER_CREDENTIALS_USR"
                // sh "echo $DOCKER_CREDENTIALS_PSW"

                echo 'Building......'
                sh 'mvn clean package -DskipTests=True'
            }

        }
        
        stage ('Docker Build') {
            steps {
                // prepare docker build context
                sh 'pwd'
                sh 'ls -lrt'
                
                // Build and push image with Jenkins' docker-plugin
                script {
                    echo 'Start - Build and push image with Jenkins docker-plugin'
                    withDockerServer([uri: "127.0.0.1:4243"]) {
                        withDockerRegistry([credentialsId: 'docker-registry-credentials', url: "https://registry-secured-dev.apps.dev.cloudsprint.io"]) {
                            // we give the image the same version as the .war package
                            def image = docker.build("registry-secured-dev.apps.dev.cloudsprint.io/Test-docker", "--build-arg PACKAGE_VERSION=1.1 ./")
                            image.push()
                            echo 'Done pushing the image.'
                        }
                    }
                }
            }
        }
          
          
        stage ('Dev_Deployment') {
            steps{
                sh 'pwd'
                sh 'which cf'
                sh 'cf --version'
                sh 'cf login -a api.sys.dev.cloudsprint.io -u $CF_CREDENTIALS_USR -p $CF_CREDENTIALS_PSW'
                sh 'cf t -o FEGO-demo -s FEGO-demo'
                sh 'CF_DOCKER_PASSWORD=$DOCKER_CREDENTIALS_PSW cf push'
              }
          }
    }
}
