pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps {
                // Obtener código de la rama
                git branch: 'master', url: 'https://github.com/Izangildev/todo-list-aws-UNIR.git'
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env production --no-fail-on-empty-changeset
                '''
            }
        }
        
        stage('Rest tests') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks \
                      --stack-name todo-list-aws-production \
                      --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                      --region us-east-1 \
                      --output text)
                    
                    pytest --junitxml=result-rest.xml -m read test/integration/todoApiTest.py
                '''
                
                junit 'result-rest.xml'
            }
        }
    }
}