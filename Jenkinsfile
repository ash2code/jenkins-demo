pipeline {
    agent {
        label 'ec2-worker'
    }

    tools {
        maven 'maven-3'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:ash2code/jenkins-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-token') {
                    sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                        -Dsonar.projectKey=java-demo \
                        -Dsonar.projectName=java-demo
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',
                        passwordVariable: 'DOCKER_PASSWORD',
                        usernameVariable: 'DOCKER_USERNAME'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ash2code/jenkins-cal:latest .
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    docker tag ash2code/jenkins-cal:latest \
                    ash2code/jenkins-cal:$BUILD_NUMBER
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push ash2code/jenkins-cal:$BUILD_NUMBER
                '''
            }
        }
    }

    post {
        success {
            echo 'CI Pipeline succeeded!'
        }

        failure {
            echo 'CI Pipeline failed!'
        }
    }
}
