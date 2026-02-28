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
                sh '''
                pip3 install flake8 bandit

                flake8 src --statistics --tee --output-file flake8-report.txt
                bandit -r src -f txt -o bandit-report.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                sam build
                sam validate

                sam deploy \
                --stack-name $STACK_NAME \
                --capabilities CAPABILITY_IAM \
                --no-confirm-changeset \
                --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                API_URL=$(aws cloudformation describe-stacks \
                --stack-name $STACK_NAME \
                --query "Stacks[0].Outputs[?OutputKey=='HelloWorldApi'].OutputValue" \
                --output text)

                echo "API URL: $API_URL"

                curl -X GET $API_URL/todo
                '''
            }
        }

        stage('Promote') {
            steps {
                sh '''
                git checkout master
                git merge develop
                git push origin master
                '''
            }
        }
    }
}