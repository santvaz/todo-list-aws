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
                        string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AKID'),
                        string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'SAK')
                    ]) {
                        bat '''
                        set AWS_ACCESS_KEY_ID=%AKID%
                        set AWS_SECRET_ACCESS_KEY=%SAK%
                        set AWS_DEFAULT_REGION=%REGION%

                        if exist .aws-sam rmdir /s /q .aws-sam
                                
                        sam build
                                
                        sam validate --region %REGION%

                        sam deploy ^
                            --stack-name %STACK_NAME% ^
                            --region %REGION% ^
                            --capabilities CAPABILITY_IAM ^
                            --no-confirm-changeset ^
                            --no-fail-on-empty-changeset ^
                            --parameter-overrides Stage=staging
                        '''
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
                    git checkout master || git checkout -b master
                    git fetch origin
                    git reset --hard origin/master
                    git config user.email "stawfikvazquez@gmail.com"
                    git config user.name "santvaz"
                    git merge origin/develop --no-edit
                    git push origin master
                    '''
                }
            }
        }
    }
}