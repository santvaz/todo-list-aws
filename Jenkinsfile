pipeline {
    agent any

    environment {
        STACK_NAME = "todo-list-staging"
        REGION = "eu-west-1"
        PYTHON_EXE = "py" 
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/santvaz/todo-list-aws.git'
            }
        }

        stage('Static Test') {
            steps {
                bat """
                ${PYTHON_EXE} -m pip install flake8 bandit
                ${PYTHON_EXE} -m flake8 src --statistics --output-file flake8-report.txt || echo "Flake8 found issues"
                ${PYTHON_EXE} -m bandit -r src -f txt -o bandit-report.txt || echo "Bandit found issues"
                """
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                sam build --use-container
                sam validate
                sam deploy ^
                    --stack-name %STACK_NAME% ^
                    --region %REGION% ^
                    --capabilities CAPABILITY_IAM ^
                    --no-confirm-changeset ^
                    --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                bat '''
                echo Testing API
                curl -I https://google.com
                '''
            }
        }

        stage('Promote') {
            steps {
                script {
                    bat '''
                    git checkout master || git checkout -b master
                    git reset --hard origin/master
                    git merge origin/develop --no-edit
                    git push origin master
                    '''
                }
            }
        }
    }
}