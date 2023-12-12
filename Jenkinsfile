pipeline{
    agent any
    tools{
        maven 'MVN'
        jdk 'JDK11'
        git 'git'
    }
    environment{
        DOCKER_LOGIN_NAME = 'akashz'
        DOCKER_PASSWORD = credentials('docker_token')
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages{
        stage('Fetch Code'){
            steps{
                git branch: 'main', url: 'https://github.com/akashzakde/spring-boot-app.git'
            }
        }
        stage('Code Analysis'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonarqube-creds',installationName: 'sonarqube') {
                sh '''mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=Java-Project \
                    -Dsonar.projectName='Java-Project' \
                    -Dsonar.host.url=http://172.31.4.11:9000 \
                    -Dsonar.token=sqp_156397e53f4007d40e1a790c9ffebbe0442cace1'''
                    }
                }
            }
        stage('QualityGate Result'){
            steps {
                    timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                    }
                }
            }
        stage('Build Code'){
            steps{
                sh 'mvn clean -DskipTests package'
            }
        }
        stage('Test Code'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Docker Image Build'){
            steps{
                sh '''
                    docker build -t $DOCKER_LOGIN_NAME/$IMAGE_NAME .
                    docker tag $DOCKER_LOGIN_NAME/$IMAGE_NAME $DOCKER_LOGIN_NAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
            }
        }
        stage('Push Image'){
            steps{
                sh '''
                    docker login -u $DOCKER_LOGIN_NAME -p $DOCKER_PASSWORD
                    docker push $DOCKER_LOGIN_NAME/$IMAGE_NAME:latest
                    docker push $DOCKER_LOGIN_NAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
            }
        }
        stage('Clean Images'){
            steps{
                sh '''
                docker rmi $DOCKER_LOGIN_NAME/$IMAGE_NAME:latest
                docker rmi $DOCKER_LOGIN_NAME/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
        stage('Trigger CD Pipeline'){
            steps{
                sh "curl -v -k --user admin:11bf95d3b7de2e69bb16e978e2825ac77c -X POST -H 'cache-control: no-cache' -H 'content_type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://65.2.172.158:8080/job/Infra-CD-Pipeline/buildWithParameters?token=Jenkins-CD-Token'"
            }
        }
    }
}
