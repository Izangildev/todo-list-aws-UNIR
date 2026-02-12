pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps {
                // Obtener codigo de la rama develop
                git branch: 'develop', url: 'https://github.com/Izangildev/todo-list-aws-UNIR.git'
            }
        }
        
        stage('Static tests') {
            steps {
                sh '''
                    flake8 --exit-zero --format=pylint src > flake8.out
                    bandit -r src > bandit.out || true
                '''
                
                 recordIssues tools: [
                    flake8(name: 'Flake8', pattern: 'flake8.out')
                ]
                
                recordIssues tools: [
                    pyLint(name: 'Bandit', pattern: 'bandit.out')    
                ]
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env staging --no-fail-on-empty-changeset
                '''
            }
        }
        
        stage('Rest tests') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks \
                      --stack-name todo-list-aws-staging \
                      --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                      --region us-east-1 \
                      --output text)
                    
                    pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                '''
                
                junit 'result-rest.xml'
            }
        }
        
        stage('Promote') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-token',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                        git config merge.ours.driver true
                        git config user.email "jenkins@local"
                        git config user.name "Jenkins"
        
                        git checkout master
                        git pull origin master
                        git merge develop
        
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/Izangildev/todo-list-aws-UNIR.git master
                    '''
                }
            }
        }