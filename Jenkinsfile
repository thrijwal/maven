pipeline {
    agent none

    environment {
        MAVEN_HOME = "/opt/maven"
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
            agent { label 'maven' }  // All nested stages share this agent
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
                stage('Deploy to GitLab') {
                    steps {
                        echo "Deploying Maven artifact to GitLab..."
                        withCredentials([usernamePassword(credentialsId: 'GITLAB_DEPLOY', 
                                                            usernameVariable: 'GITLAB_DEPLOY_USERNAME', 
                                                            passwordVariable: 'GITLAB_DEPLOY_TOKEN')]) {
                            // Run Maven deploy goal; distributionManagement in pom.xml should point to GitLab
                            sh "${MAVEN_HOME}/bin/mvn deploy"
                        }
                    }
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
