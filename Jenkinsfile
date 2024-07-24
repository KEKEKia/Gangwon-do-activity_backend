pipeline {
    agent any
    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-17.0.9.0.9-2.el8_8.x86_64'
        PATH = "${JAVA_HOME}/bin:/usr/bin:${env.PATH}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo '코드 체크아웃 중...'
                git branch: 'main', url: 'https://github.com/K-srand/Gangwon-do-activity_backend.git'
            }
        }
        stage('Grant Permissions') {
            steps {
                echo 'gradlew에 실행 권한 부여 중...'
                sh 'chmod +x ./gradlew'
            }
        }
        stage('Build') {
            steps {
                echo '프로젝트 빌드 중...'
                sh 'java -version'  // Java 버전 확인
                sh './gradlew build'
            }
        }
        stage('Docker Build') {
            steps {
                echo 'Docker 빌드 준비 중...'
                script {
                    sh 'docker buildx version' // Docker Buildx가 설치되었는지 확인
                    withCredentials([usernamePassword(credentialsId: 'SPRING_MAIL_CREDENTIALS', usernameVariable: 'SPRING_MAIL_USERNAME', passwordVariable: 'SPRING_MAIL_PASSWORD'),
                                     string(credentialsId: 'AWS_ACCESS_KEY', variable: 'AWS_ACCESS_KEY'),
                                     string(credentialsId: 'AWS_SECRET_KEY', variable: 'AWS_SECRET_KEY')]) {
                       sh '''
                           /usr/bin/docker buildx build --progress=plain -t backend-app:latest --build-arg SPRING_MAIL_USERNAME=$SPRING_MAIL_USERNAME --build-arg SPRING_MAIL_PASSWORD=$SPRING_MAIL_PASSWORD --build-arg AWS_ACCESS_KEY=$AWS_ACCESS_KEY --build-arg AWS_SECRET_KEY=$AWS_SECRET_KEY -f Dockerfile .
                       '''

                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo '애플리케이션 배포 중...'
                script {
                    sh '/usr/bin/docker stop backend-app || true'
                    sh '/usr/bin/docker rm backend-app || true'
                    sh '/usr/bin/docker run -d -p 4040:4040 --name backend-app backend-app:latest'
                    echo "Docker 컨테이너가 성공적으로 시작되었습니다."
                }
            }
        }
    }
}
