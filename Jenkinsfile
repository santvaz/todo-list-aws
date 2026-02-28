pipeline{
    agent any

    environment {
        REPO_URL = "https://github.com/santvaz/todo-list-aws.git"
    }

    stages{

        stage('Get Code'){
            steps{
                git branch: 'develop', url: "${REPO_URL}"
            }
        }

        stage('Static Test'){
            steps{
                script{

                    bat """
                    python -m pip install flake8 bandit
                    python -m flake8 --exit-zero --format=pylint src > flake8.out
                    """

                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [
                        [threshold: 8, type: 'TOTAL', unstable: true],
                        [threshold: 10, type: 'TOTAL', unstable: false]
                    ]

                    bat "del flake8.out"

                    bat """
                    python -m bandit -r src -o bandit.out -f custom --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                    """

                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [
                        [threshold: 2, type: 'TOTAL', unstable: true],
                        [threshold: 4, type: 'TOTAL', unstable: false]
                    ]

                    bat "del bandit.out"

                }
            }
        }

        stage('Deploy'){
            steps{
                script{

                    bat """
                    sam deploy ^
                        --stack-name todo-list-aws-staging ^
                        --region us-east-1 ^
                        --capabilities CAPABILITY_IAM ^
                        --no-confirm-changeset ^
                        --no-fail-on-empty-changeset
                    """

                    def stackOutputs = bat(
                        script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query \"Stacks[0].Outputs\" --output json",
                        returnStdout: true
                    ).trim()

                    echo "Stack outputs: ${stackOutputs}"

                    env.BASE_URL = "URL_FROM_OUTPUT"

                }
            }
        }

        stage('Rest Test'){
            steps{

                bat """
                python -m pip install pytest requests
                python -m pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                """

                junit 'result-rest.xml'

                bat "del result-rest.xml"
            }
        }

        stage('Promote'){
            steps{
                script{

                    withCredentials([usernamePassword(credentialsId: 'git-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){

                        bat "git config user.email santvaz@example.com"
                        bat "git config user.name santvaz"

                        bat "git add -A"
                        bat "git commit -m CI"

                        bat "git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/%GIT_USERNAME%/todo-list-aws.git"

                        bat "git checkout master"

                        bat "git merge develop -m \"Subiendo cambios a Master\""

                        bat "git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/%GIT_USERNAME%/todo-list-aws.git"

                    }

                }
            }
        }

    }

    post{
        always{
            echo "Cleaning workspace"
            deleteDir()
        }
    }
}