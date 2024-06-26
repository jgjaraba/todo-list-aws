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
                git branch: 'develop', credentialsId: 'github_01', url: 'https://github.com/jgjaraba/todo-list-aws.git'
                wget https://raw.githubusercontent.com/jgjaraba/todo-list-aws_config/staging/samconfig.toml
            }
        }
        stage("Static Test"){
            steps{
                sh '''
                    echo $PATH
                    flake8 --exit-zero --format=pylint src >flake8.out
                    bandit --exit-zero -r  ./src -f custom -o bandit.out --severity-level medium --msg-template="{relpath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        stage("Deploy"){
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name todo-list-aws-staging --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-8ojqf2mgirsb --s3-prefix todo-list-aws --region us-east-1 --capabilities CAPABILITY_IAM --parameter-overrides Stage=staging --no-fail-on-empty-changeset
                    aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query "Stacks[0].Outputs[?OutputKey == 'BaseUrlApi'].OutputValue | [0]" > baseUrl.log
                    sed -i 's/"//g' baseUrl.log
                '''
            }
        }
        stage("Rest Test"){
            steps{
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    sh '''
                        export BASE_URL=$(cat baseUrl.log)
                        pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                    '''
                    junit 'result-rest.xml'
                }
            }
        }
        stage("Promote"){
            steps{
                withCredentials([ gitUsernamePassword(credentialsId: 'github_01', gitToolName: 'Default')]) {
                    sh '''
                        git checkout master
                        git merge --no-ff --no-commit develop
                        git reset HEAD Jenkinsfile
                        git checkout -- Jenkinsfile
                        git commit -m "merged develop branch"
                        git push origin master
                    '''
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
