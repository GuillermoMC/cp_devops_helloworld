pipeline {
    agent any

    stages {
        
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