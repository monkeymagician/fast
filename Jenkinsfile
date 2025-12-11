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
