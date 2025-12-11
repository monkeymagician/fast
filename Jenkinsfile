pipeline {
    agent any

    environment {
        TIME_ZONE = 'Asia/Seoul'
        
        // [Fast App 리포지토리] (빌드용)
        GIT_TARGET_BRANCH = 'main'
        GIT_REPOSITORY_URL = 'https://github.com/monkeymagician/fast.git'
        GIT_CREDENTIONALS_ID = 'git_cre'

        // [Deployment 리포지토리] (배포 파일 수정용)
        // ★ 중요: 여기에 test-dep.yml 파일이 있어야 합니다.
        GIT_REPOSITORY_DEP = 'https://github.com/monkeymagician/deployment.git' 
        
        GIT_EMAIL = '221csw2@gmail.com'
        GIT_NAME = 'monkeymagician'

        // AWS ECR
        AWS_ECR_CREDENTIAL_ID = 'aws_cre'
        AWS_ECR_URI = '651109015678.dkr.ecr.ap-northeast-2.amazonaws.com'
        AWS_ECR_IMAGE_NAME = 'fast'
        AWS_REGION = 'ap-northeast-2'
    }

    stages {
        stage('1.init') {
            steps {
                echo '1.init stage'
                deleteDir()
            }
        }

        stage('2.Cloning Repository') {
            steps {
                // 여기서는 fast 리포지토리를 가져옵니다 (소스 코드)
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

        // [여기가 핵심입니다]
        stage('5.EKS manifest file update') {
            steps {
                // 1. 기존 fast 리포지토리 내용은 지우고 deployment 리포지토리를 새로 가져옵니다.
                // deleteDir() // 안전하게 지우고 시작 (선택사항)
                
                // 2. Deployment 리포지토리 체크아웃 (주소 변경됨)
                git branch: 'main',
                    credentialsId: "${GIT_CREDENTIONALS_ID}",
                    url: "${GIT_REPOSITORY_DEP}" // <-- deployment.git으로 접속
                
                script {
                    sh '''
                    echo "Current Directory Check:"
                    ls -al  # test-dep.yml이 있는지 로그로 확인

                    git config --global user.email ${GIT_EMAIL}
                    git config --global user.name ${GIT_NAME}
                    
                    # 3. 이제 deployment 리포지토리 안에 있는 파일을 수정합니다.
                    sed -i "s@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:.*@${AWS_ECR_URI}/${AWS_ECR_IMAGE_NAME}:${BUILD_NUMBER}@g" test-dep.yml
                    
                    git add .
                    git branch -M main
                    git commit -m "fixed tag ${BUILD_NUMBER}"
                    
                    # 4. deployment 리포지토리에 푸시
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
