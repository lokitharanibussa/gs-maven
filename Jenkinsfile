pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_URL = 'https://13.233.245.91:8081/repository/maven-releases/'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'  // Jenkins credential ID for Nexus
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}:8080/manager/text/deploy?path=/gs-maven&update=true"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rama3058/gs-maven.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    dir('complete') {
                        // Using usernamePassword for SonarQube credentials
                        withCredentials([usernamePassword(credentialsId: 'sonar', usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_TOKEN')]) {
                            sh """
                                mvn clean verify sonar:sonar \
                                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                    -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                                    -Dsonar.host.url=${SONAR_HOST_URL} \
                                    -Dsonar.login=${SONAR_TOKEN}
                            """
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Ensure that you're in the correct directory
                    dir('complete') {
                        sh 'mvn clean package'
                        archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Define the artifact file (assuming it's in the target directory)
                    def artifactFile = 'target/gs-maven-0.1.0-SNAPSHOT.jar'
                    echo "Uploading artifact ${artifactFile} to Nexus"
                    
                    // Use Jenkins credentials securely with the 'withCredentials' block
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        sh """
                            mvn deploy:deploy-file \
                                -Dfile=${artifactFile} \
                                -DrepositoryId=nexus \
                                -Durl=${NEXUS_URL} \
                                -DgroupId=com.example \
                                -DartifactId=gs-maven \
                                -Dversion=0.1.0-SNAPSHOT \
                                -Dpackaging=jar \
                                -Dusername=${NEXUS_USERNAME} \
                                -Dpassword=${NEXUS_PASSWORD}
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = 'target/gs-maven-0.1.0-SNAPSHOT.war'
                    echo "Deploying ${warFile} to Tomcat"
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} --upload-file ${warFile} ${TOMCAT_DEPLOY_URL}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
