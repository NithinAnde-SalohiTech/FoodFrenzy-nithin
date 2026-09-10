pipeline {
    agent any

    environment {
        APP_EC2_IP = '172.31.10.254'
    }

    tools {
        jdk 'JAVA-17'
        maven 'MAVEN'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/NithinAnde-SalohiTech/FoodFrenzy-nithin.git',
                    branch: 'master'
            }
        }

        stage('Validate') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Sonar Scan') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'SONAR_ID',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    withSonarQubeEnv('sonarqube') {
                        sh '''
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.6.0.6792:sonar \
                                -Dsonar.projectKey=nithinande-salohitech \
                                -Dsonar.organization=nithinande-salohitech \
                                -Dsonar.host.url=https://sonarcloud.io \
                                -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }
        stage('Check Maven Output') {
            steps {
                sh '''
                    echo "===== WORKSPACE ====="
                    pwd
                    echo "===== TARGET DIRECTORIES ====="
                    find . -type d -name target -print
                    echo "===== ALL JAR FILES ====="
                    find . -type f -name "*.jar" -print
                '''
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t nithinandedocker/test-frenzy:latest .
                '''
            }
        }
        
        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'DOCKER_ID',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push nithinandedocker/test-frenzy:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'APP_EC2_SSH',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP_EC2_IP" \
                            "
                                docker pull nithinandedocker/test-frenzy:latest &&
                                docker stop test-frenzy || true &&
                                docker rm test-frenzy || true &&
                                docker run -d \
                                    --name test-frenzy \
                                    -p 8081:8081 \
                                    nithinandedocker/test-frenzy:latest
                            "
                    '''
                }
            }
        }
    }
}
