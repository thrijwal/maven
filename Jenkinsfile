pipeline {
    agent none

    triggers {
        githubPush()
    }

    environment {
        MAVEN_HOME = "/opt/maven"
        // Define the JFrog repository URL here
        JFROG_REPO = "https://trial3j8up4.jfrog.io/artifactory/my-maven-v"
    }

    stages {
        stage('Setup Ansible Playbook') {
            agent { label 'ansible' }
            steps {
                echo "Checking out Ansible repository..."
                checkout([$class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: 'https://github.com/thrijwal/My_Ansible_Projects.git']]
                ])
                echo "Running Ansible playbook to set up Maven environment on Jnode1 and Jnode2..."
                sh "cd Ansible_Jenkins && ansible-playbook -i inventory maven_playbook.yml"
            }
        }

        stage('Maven Pipeline') {
            agent { label 'maven' }
            stages {
                stage('Checkout Maven Project') {
                    steps {
                        echo "Checking out Maven repository..."
                        checkout([$class: 'GitSCM', 
                            branches: [[name: 'main']], 
                            userRemoteConfigs: [[url: 'https://github.com/thrijwal/maven.git']]
                        ])
                        // List files to verify checkout
                        sh "ls -la"
                    }
                }
                stage('Build') {
                    steps {
                        echo "Building Maven project..."
                        sh "${MAVEN_HOME}/bin/mvn clean package"
                    }
                }
                stage('Test') {
                    steps {
                        echo "Running tests..."
                        sh "${MAVEN_HOME}/bin/mvn test"
                    }
                }
                stage('SonarQube Analysis') {
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
