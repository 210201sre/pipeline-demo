pipeline {
    // Author: Matthew Oberlies
    agent any
    environment {
        // Can declare different environment variables to be used specifically within this pipeline
        DOCKER_IMAGE_NAME = 'ikenoxamos/project1'
        MAVEN_IMAGE_NAME = 'log-aggregation-demo:0.0.1-SNAPSHOT'
    }

    stages {

        stage('Build') {
            steps {
                sh 'chmod +x mvnw && ./mvnw spring-boot:build-image'
                sh 'docker tag $MAVEN_IMAGE_NAME $DOCKER_IMAGE_NAME'
                script {
                    app = docker.image(DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Sonar Quality Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonarcloud') {
                    sh './mvnw -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                script {
                    timeout(time: 30, unit: 'MINUTES') {
                        qualitygate = waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Push Docker Image') {
            // According to the documentation, this branch instruction only works
            // with the Multi-Branch Pipeline Jenkins Job type
            // This particular one, is only the Pipeline type  
            // when {
            //     // Only execute the Docker Push stage if we are on the master branch
            //     branch 'master'
            // }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-jenkins-token') {
                        // These are the image tags
                        app.push('latest')
                        // Double quotes will interpolate environment variables in Groovy
                        app.push("${env.BUILD_NUMBER}")
                        app.push("${env.GIT_COMMIT}")
                    }
                }
            }
        }

        stage('Canary Deployment') {
            environment {
                CANARY_REPLICAS = 1
                // This will represent the number of canary instances that we will create
                // Our Service that is configured in the cluster already will include this
                // this canary instance when it routes traffic

                // For example, if our original deployment had 4 replicas
                // And we create 1 canary replica

                // Our canary will receive 20% of the total traffic (1 out of 5)
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig-sre',
                    // This is the ID of the kubeconfig credentials we made
                    enableConfigSubstitution: true,
                    // Is a very convenient property
                    // Allows our kubernetes manifests to interpolate information
                    // from our environment variables
                    configs: 'manifests/canary-deployment.yml'
                )
            }
        }

        stage('Production Deployment') {
            environment {
                CANARY_REPLICAS = 0
            }

            steps {
                // Some "button press"
                input 'Deploy to Production?'
                // if(response == 'NO') {

                // }
                // Effectively a yes/no prompt in Jenkins
                // milestone(1) // Comes from a separate plugin
                // Would help with many builds happening before the previous one finishes
                // If a new version/build passes an older version/build, then just discard the older one

                // Scale down the canary
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig-sre',
                    enableConfigSubstitution: true,
                    configs: 'manifests/canary-deployment.yml'
                )

                // Scale up the new production version
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig-sre',
                    enableConfigSubstitution: true,
                    configs: 'manifests/production-deployment.yml'
                )
            }
        }
    }

    // post {
    //     always {
            // Can use the previously created qualitygate variable
            // Perhaps include the results as part of a discordSend instruction
            // Might use another "script" scope
    //     }
    // }
}