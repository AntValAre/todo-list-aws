pipeline {
    agent any 

    stages {
        stage('Get Code') {
            steps {
                deleteDir()
                git branch: 'master', url: 'https://github.com/AntValAre/todo-list-aws.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy --config-env production --no-confirm-changeset --resolve-s3 --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test Production') {
            steps {
              
                    sh '''

                        export BASE_URL=$(aws cloudformation describe-stacks \
                            --region us-east-1 \
                            --stack-name todo-list-aws-production \
                            --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                            --output text)

                        
                        pytest -s -v -m read_only --junitxml=rest_report_deploy.xml test/integration/todoApiTest.py
                    '''
                    junit 'rest_report_deploy.xml'
                }
            }
        }
    }
