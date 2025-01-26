pipeline {
    agent any

    stages {
        
        stage('GetCode') {
            
            steps {
                
                git 'https://github.com/GuillermoMC/cp_devops_helloworld' 
                
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
                                coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                             '''

                            stash name: 'testunit', includes: 'result-unit.xml'
                            stash name: 'coverunit', includes: 'coverage.xml'
                            stash name: 'coverage-data': includes: '.coverage' 

                        }
                        
                    }
                    
                }
                
                // wiremock lo tengo levantado en un contenedor docker
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
        
        stage('Static') {
            
            steps {
                
                bat '''
                    flake8 --exit-zero --format=pylint app > flake8.out
                '''
                
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            
            }
            
        }
        
        stage('Security') {
            
            steps {
                
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                
            }
            
        }
        
        stage('Coverage') {
            
            steps {
                
                unstash name: 'coverage-data'

                bat '''
                    coverage xml -o coverage.xml
                '''

                cobertura coberturaReportFile: 'coverage.xml', lineCoverageTargets: '95,0,85', conditionalCoverageTargets: '90,0,80' 
                
            }
            
        }
        
        stage('Performance') {
            
            steps {
                
                bat '''
                     C:\\Users\\Guille\\Desktop\\apache-jmeter-5.6.3\\bin\\jmeter -n -t C:\\Users\\Guille\\Desktop\\apache-jmeter-5.6.3\\bin\\CP1-2.jmx -f -l flask.jtl
                '''
                
                perfReport sourceDataFiles: 'flask.jtl'
                
            }
            
        }
 
        stage('Limpieza') {
            
            agent any
            
            steps {
                
                unstash name: 'testunit'
                unstash name: 'coverunit'
                unstash name: 'coverage-data'
                cleanWs()
                
            }
            
        }

    }


    
}