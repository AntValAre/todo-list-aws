pipeline {
    agent any 
    
    post {
        always {
            deleteDir()
        }
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/AntValAre/Practica1.4.git'
            }
        }

        stage('Static Test') {
            steps {
                sh '''

                    flake8  src/ --output-file=flake8_report.txt 
                    
                    bandit  -r src/ -f txt -o bandit_report.txt 
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'flake8_report.txt, bandit_report.txt', followSymlinks: false
                }
            }
        }
        
        stage('Deploy Staging') {
            steps {
                
                    sh '''
                        
                        sam build
                        sam validate --region us-east-1
                        sam deploy --config-env staging --no-confirm-changeset --resolve-s3 --no-fail-on-empty-changeset
                    '''
                
            }
        }
        stage('Rest Test') {
            steps {
                
                    sh '''
                        
                        export BASE_URL=$(aws cloudformation describe-stacks \
                            --region us-east-1 \
                            --stack-name staging-todo-list-aws \
                            --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                            --output text)

                        
                       pytest -s -v --junitxml=rest_report.xml test/integration/todoApiTest.py
                        '''

                        junit 'rest_report.xml'
                }
                post {
                always {
                    archiveArtifacts artifacts: 'rest_report.xml', followSymlinks: false
                }
            }
        }
        stage('Promote') {
            when {
                equals expected: 'SUCCESS', actual: currentBuild.currentResult
            }
            steps {
    
                withCredentials([usernamePassword(credentialsId: 'GitHub-PAT-Cred', passwordVariable: 'GIT_TOKEN', usernameVariable: 'GIT_USERNAME')]) {
                    sh '''
                        git remote set-url origin https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/AntValAre/Practica1.4.git
                        git fetch origin
                        git checkout master
                        git reset --hard origin/master
                        git merge origin/develop --no-commit --no-ff
                        git checkout HEAD -- Jenkinsfile
                        git commit -m "Promote: Merge develop a master conservando Jenkinsfile de CD"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/AntValAre/Practica1.4.git master
                    '''
                }
            }
        }
    }
}
