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
    }
}