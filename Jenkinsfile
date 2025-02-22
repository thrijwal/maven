pipeline {
    agent { label 'Jen_node1' }  // Run on Jen_node1
    environment {
        MAVEN_HOME = "/opt/maven"  // Set Maven home
    }
    stages {
        stage('Checkout') {
            steps {
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
    }
    post {
        success {
            echo "Build and Tests executed successfully!"
        }
        failure {
            echo "Build or Tests failed!"
        }
    }
}
