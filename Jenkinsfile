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
                            /* Lanzar FLASK */
                            bat '''

                                SET FLASK_APP=app\\api.py
                                start flask run
                                
                            '''
                            /* Esperar 2 minutos como máximo a que el servicio flask este operativo */
                            timeout(2) {
                                waitUntil {
                                    script {
                                        try {
                                            def respuesta = httpRequest 'http://localhost:5000/'
                                            return (respuesta.status == 200)
                                        }
                                        catch (exception) {
                                            return false
                                        }
                                    }
                                }
                            }
                            /* Pruebas REST */
                            bat '''
                              
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