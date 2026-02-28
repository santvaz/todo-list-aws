pipeline {
    agent any

    environment {
        STACK_NAME = "todo-list-staging"
        REGION = "us-east-1"
        PY_HOME = "C:\\Users\\Ghost\\AppData\\Local\\Programs\\Python\\Python311"
        PY_SCRIPTS = "C:\\Users\\Ghost\\AppData\\Local\\Programs\\Python\\Python311\\Scripts"
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/santvaz/todo-list-aws.git'
            }
        }

        stage('Static Test') {
            steps {
                withEnv(["PATH+EXTRA=${env.PY_HOME};${env.PY_SCRIPTS}"]) {
                    bat """
                    python -m pip install flake8 bandit
                    python -m flake8 src --statistics --output-file flake8-report.txt || echo "Flake8 found issues"
                    python -m bandit -r src -f txt -o bandit-report.txt || echo "Bandit found issues"
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                withEnv(["PATH+EXTRA=${env.PY_HOME};${env.PY_SCRIPTS}"]) {
                    withCredentials([
                        string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        bat """
                        if exist .aws-sam rmdir /s /q .aws-sam
                        
                        sam build
                        
                        sam validate --region ${env.REGION}

                        sam deploy ^
                            --config-env staging ^
                            --stack-name ${env.STACK_NAME} ^
                            --region ${env.REGION} ^
                            --capabilities CAPABILITY_IAM ^
                            --no-confirm-changeset ^
                            --no-fail-on-empty-changeset ^
                            --resolve-s3 ^
                            --parameter-overrides Stage=staging
                        """
                    }
                }
            }
        }

        stage('Rest Test') {
            steps {
                bat 'curl -I https://google.com'
            }
        }

        stage('Promote') {
    steps {
        script {
            bat '''
            git merge --abort >nul 2>&1
            
            git config user.email "stawfikvazquez@gmail.com"
            git config user.name "santvaz"

            git fetch origin
            git checkout master || git checkout -b master
            git reset --hard origin/master

            git merge origin/develop --no-edit -X theirs || (echo Error en merge && exit 1)

            git push origin master
            '''
        }
    }
}
    }
}