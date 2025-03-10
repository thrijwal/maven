@Library('jenkins-shared-library') _

pipeline {
    agent none

    triggers {
        githubPush()
    }

    environment {
        MAVEN_HOME = "/opt/maven"
        JFROG_REPO = "https://trial3j8up4.jfrog.io/artifactory/my-maven-v"
    }

    stages {
        stage('Checkout Maven Project') {
            agent { label 'maven' }
            steps {
                echo "Checking out Maven repository..."
                checkout([$class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: 'https://github.com/thrijwal/maven.git']]])
                sh "ls -la"
            }
        }
        stage('Build') {
            agent { label 'maven' }
            steps {
                mavenBuild(MAVEN_HOME)
            }
        }
        stage('Test') {
            agent { label 'maven' }
            steps {
                mavenTest(MAVEN_HOME)
            }
        }
        stage('SonarQube Analysis') {
            agent { label 'maven' }
            steps {
                echo "Running SonarQube analysis..."
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh "${MAVEN_HOME}/bin/mvn sonar:sonar " +
                       "-Dsonar.projectKey=thrijwal " +
                       "-Dsonar.organization=thrijwal " +
                       "-Dsonar.host.url=https://sonarcloud.io " +
                       "-Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }
        stage('Deploy to GitLab') {
            agent { label 'maven' }
            steps {
                echo "Deploying Maven artifact to GitLab..."
                withCredentials([usernamePassword(credentialsId: 'GITLAB_DEPLOY', 
                                                    usernameVariable: 'GITLAB_DEPLOY_USERNAME', 
                                                    passwordVariable: 'GITLAB_DEPLOY_TOKEN')]) {
                    sh "${MAVEN_HOME}/bin/mvn deploy"
                }
            }
        }
        stage('Deploy to JFrog Artifactory') {
            agent { label 'maven' }
            steps {
                echo "Deploying Maven artifact to JFrog Artifactory..."
                withCredentials([string(credentialsId: 'JFROG_CREDENTIALS', variable: 'JFROG_TOKEN')]) {
                    sh "${MAVEN_HOME}/bin/mvn deploy " +
                       "-DaltDeploymentRepository=jfrog-maven::default::${JFROG_REPO} " +
                       "-DrepositoryId=jfrog-maven " +
                       "-Dtoken=${JFROG_TOKEN}"
                }
            }
        }
    }

    post {
        success {
            echo "Build, Tests, SonarQube Analysis, and Deployment to GitLab and JFrog succeeded!"
        }
        failure {
            echo "Build, Tests, SonarQube Analysis, or Deployment failed!"
        }
    }
}
