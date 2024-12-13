pipeline {
    agent any

    stages {
        
        stage('GetCode') {
            steps {
                git 'https://github.com/GuillermoMC/cp_devops_helloworld' 
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building'
                bat '''
                    dir
                '''
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                echo %WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                             '''
                        }
                    }
                }
                
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                SET FLASK_APP=app\\api.py
                                start flask run
                               
                                set PYTHONPATH=%WORKSPACE%
                                echo %WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                            
                        }
                    }
                }
            }
        }

        stage('Results') {
            steps {
                junit 'result*.xml'
            }
        }
        
    }
}