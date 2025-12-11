pipeline {
    agent any

  environment {
        TIME_ZONE = 'Asia/Seoul'
        
        // GitHub 계정정보. 본인껄로 넣으세요!!
        GIT_TARGET_BRANCH = 'main'
        GIT_REPOSITORY_URL = 'https://github.com/monkeymagician/fast'
        GIT_CREDENTIONALS_ID = 'git_cre'


        // AWS ECR
        AWS_ECR_CREDENTIAL_ID = 'aws_cre'
        AWS_ECR_URI = '651109015678.dkr.ecr.ap-northeast-2.amazonaws.com'
        AWS_ECR_IMAGE_NAME = 'fast'
        AWS_REGION = 'ap-northeast-2'
        
    }

    stages {

        stage('init') {
            steps {
                echo 'init stage'
                deleteDir()
            }
        }

        stage('Cloning Repository') {
            steps {
                echo 'Cloning Repository'
                git branch: "${GIT_TARGET_BRANCH}",
                    credentialsId: "${GIT_CREDENTIONALS_ID}",
                    url: "${GIT_REPOSITORY_URL}"
            }
        }


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
                // 명령어가 많아질것같아서 스크립트 블록을 추가.
                // BUILD_NUMBER = 젠킨스가 제공해주는 변수.
            }
        }

        stage('4.Push to ECR') {
            steps {
            // AWS Credential 플러그인을 설치해서 사용할 수 있는 함수. aws configure와 같음
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
        





    }
}

