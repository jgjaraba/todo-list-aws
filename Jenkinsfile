pipeline {
    agent any
    
    environment {
        PATH="/var/lib/jenkins/workspace/CP1-D-reto1/venv/bin:/var/lib/jenkins/.local/bin:$PATH"
        PYTHONPATH = "."
    }
    
    options {
        skipDefaultCheckout true
    }

    stages {
        stage("Get code") {
            steps {
                git branch: 'master', credentialsId: 'github_01', url: 'https://github.com/jgjaraba/todo-list-aws.git'
                wget https://raw.githubusercontent.com/jgjaraba/todo-list-aws_config/production/samconfig.toml
            }
        }
        stage("Deploy"){
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name todo-list-aws-production --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-8ojqf2mgirsb --s3-prefix todo-list-aws --region us-east-1 --capabilities CAPABILITY_IAM --parameter-overrides Stage=production --no-fail-on-empty-changeset
                    aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[?OutputKey == 'BaseUrlApi'].OutputValue | [0]" > baseUrl.log
                    sed -i 's/"//g' baseUrl.log
                '''
            }
        }
        stage("Rest Test"){
            steps{
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    sh '''
                        export BASE_URL=$(cat baseUrl.log)
                        pytest -m readonly --junitxml=result-rest.xml test/integration/todoApiTest.py
                    '''
                    junit 'result-rest.xml'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}