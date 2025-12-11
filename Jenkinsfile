pipeline {
    agent any

    environment {
        TIME_ZONE = 'Asia/Seoul'
        
        // GitHub 계정정보
        GIT_TARGET_BRANCH = 'main'
        GIT_REPOSITORY_URL = 'https://github.com/monkeymagician/fast.git'
        GIT_CREDENTIONALS_ID = 'git_cre'

        // [수정 1] 아래 변수 3개가 없어서 에러가 났었습니다. 추가했습니다.
        // 배포용(Manifest) 파일이 있는 레포지토리 주소 (현재 같은 레포라면 위와 동일하게)
        GIT_REPOSITORY_DEP = 'https://github.com/monkeymagician/fast.git' 
        
        // 깃 허브에 커밋할 때 기록될 이름과 이메일 (본인 것으로 수정하세요)
        GIT_EMAIL = 'monkeymagician@example.com'
        GIT_NAME = 'monkeymagician'

        // AWS ECR
        AWS_ECR_CREDENTIAL_ID = 'aws_cre'
        AWS_ECR_URI = '651109015678.dkr.ecr.ap-northeast-2.amazonaws.com'
        AWS_ECR_IMAGE_NAME = 'fast'
        AWS_REGION = 'ap-northeast-2'
    }

    stages {
        // 첫번째 스테이지 : 초기화
        stage('1.init') {
            steps {
                echo '1.init stage'
                deleteDir()
            }
        }

        // 두번째 스테이지 : 소스코드 클론
        stage('2.Cloning Repository') {
            steps {
                echo '2.Cloning Repository'
                git branch: "${GIT_TARGET_BRANCH}",
                    credentialsId: "${GIT_CREDENTIONALS_ID}",
                    url: "${GIT_REPOSITORY_URL}"
            }
        }

        stage('3.Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER} .
                        docker build -t ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('4.Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_ECR_CREDENTIAL_ID}"]]) {
                    script {
                        sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ECR_URI}
                        docker push ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:latest
                        '''
                    }
                }
            }
            post {
                failure {
                    script {
                        sh '''
                        docker rm -f ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}
                        docker rm -f ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:latest
                        echo docker image push fail
                        '''
                    }
                }
                success {
                    script {
                        sh '''
                        docker rm -f ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}
                        docker rm -f ${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:latest
                        echo docker image push success
                        '''
                    }
                }                
            }
        }

        stage('5.EKS manifest file update') {
            steps {
                // 여기서 GIT_REPOSITORY_DEP
