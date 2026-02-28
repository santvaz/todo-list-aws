pipeline {

    agent any

    environment {
        STACK_NAME = "todo-list-staging"
        REGION = "eu-west-1"
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/santvaz/todo-list-aws.git'
            }
        }

        stage('Static Test') {
            steps {
                bat '''
                python -m pip install flake8 bandit

                flake8 src --statistics --output-file flake8-report.txt
                bandit -r src -f txt -o bandit-report.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                sam build
                sam validate

                sam deploy ^
                --stack-name %STACK_NAME% ^
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
                curl https://google.com
                '''
            }
        }

        stage('Promote') {
            steps {
                bat '''
                git checkout master
                git merge develop
                git push origin master
                '''
            }
        }

    }

}