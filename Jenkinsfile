pipeline {
    agent any

    environment {
        TIME_ZONE = 'Asia/Seoul'
        
        // GitHub 계정정보
        GIT_TARGET_BRANCH = 'main'
        GIT_REPOSITORY_URL = 'https://github.com/monkeymagician/fast.git'
        GIT_CREDENTIONALS_ID = 'git_cre'

        // [수정 1] 본인 이메일 적용됨
        GIT_EMAIL = '221csw2@gmail.com'
        GIT_NAME = 'monkeymagician'
        
        // [수정 2] 주소를 SSH(git@...)에서 HTTPS로 변경하고, 'fast' 레포지토리로 통일했습니다.
        // (별도의 deployment 레포지토리가 있고 SSH키 설정을 하신 게 아니라면, 아래 주소를 쓰셔야 합니다)
        GIT_REPOSITORY_DEP = 'https://github.com/monkeymagician/fast.git'


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
                // 수정된 HTTPS 주소로 자격증명(git_cre)을 사용하여 접속
                git credentialsId: GIT_CREDENTIONALS_ID, url: GIT_REPOSITORY_DEP, branch: 'main'
                
                script {
                    // [수정 3] 여기에 'sh'가 빠져 있어서 실행이 안 됐습니다. 추가했습니다!
                    sh '''
                    git config --global user.email ${GIT_EMAIL}
                    git config --global user.name ${GIT_NAME}
                    
                    # [수정 4] sed 명령어 안의 변수가 잘 작동하도록 쌍따옴표(")로 감쌌습니다.
                    # test-dep.yml 파일이 존재하는지 꼭 확인해주세요.
                    sed -i "s@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:.*@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}@g" test-dep.yml
                    
                    git add .
                    git branch -M main
                    git commit -m "fixed tag ${BUILD_NUMBER}"
                    
                    # 리모트 재설정 (에러 방지용 || true 추가)
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
