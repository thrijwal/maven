pipeline {
    agent { label 'maven' }
    environment {
        MAVEN_HOME = "/opt/maven"
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout your project from GitHub
                git branch: 'main', url: 'https://github.com/thrijwal/maven.git'
            }
        }
        stage('Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }
        stage('Test') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn test"
            }
        }
        stage('Deploy to GitLab') {
            steps {
                // Bind deploy token credentials to environment variables
                withCredentials([usernamePassword(credentialsId: 'GITLAB_DEPLOY', 
                                                    usernameVariable: 'GITLAB_DEPLOY_USERNAME', 
                                                    passwordVariable: 'GITLAB_DEPLOY_TOKEN')]) {
                    // Optional: check effective settings to ensure substitution worked
                    sh "${MAVEN_HOME}/bin/mvn help:effective-settings"
                    
                    // Run Maven deploy goal; distributionManagement in pom.xml should point to GitLab
                    sh "${MAVEN_HOME}/bin/mvn deploy"
                }
            }
        }
    }
    post {
        success {
            echo "Build, Tests, and Deployment succeeded!"
        }
        failure {
            echo "Build, Tests, or Deployment failed!"
        }
    }
}
