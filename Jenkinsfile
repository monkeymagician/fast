pipeline {
    agent any

    environment {
        TIME_ZONE = 'Asia/Seoul'
        
        // GitHub 계정정보
        GIT_TARGET_BRANCH = 'main'
        GIT_REPOSITORY_URL = 'https://github.com/monkeymagician/fast.git'
        GIT_CREDENTIONALS_ID = 'git_cre'

        // [중요] 배포용(Manifest) 파일이 있는 레포지토리 주소
        GIT_REPOSITORY_DEP = 'https://github.com/monkeymagician/fast.git' 
        
        // [중요] 깃 허브 커밋용 이메일/이름
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
                git credentialsId: GIT_CREDENTIONALS_ID, url: GIT_REPOSITORY_DEP, branch: 'main'
                
                script {
                    // [수정 포인트] sh 명령어가 정확히 들어가 있습니다.
                    sh '''
                    git config --global user.email ${GIT_EMAIL}
                    git config --global user.name ${GIT_NAME}
                    
                    # sed 명령어로 test-dep.yml 파일의 이미지 태그 수정
                    sed -i "s@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:.*@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}@g" test-dep.yml
                    
                    git add .
                    git branch -M main
                    git commit -m "fixed tag ${BUILD_NUMBER}"
                    
                    git remote remove origin || true
                    git remote add origin ${GIT_REPOSITORY_DEP}
                    
                    git push origin main
                    '''
                }
            }
            post {
                failure {
                    sh "echo manifest update failed"
                }
                success {
                    sh "echo manifest update success"
                }
            }
        }
    }
}
