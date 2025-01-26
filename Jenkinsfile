pipeline {
    
    agent any
    
    stages {
        
        stage('GetCode') {

            steps {

                git 'https://github.com/GuillermoMC/cp_devops_helloworld' 
                bat 'dir'
                echo WORKSPACE

            }
        }
        
      
        stage('Unit') {

            steps {

                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {

                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        echo %WORKSPACE%
                        pytest --junitxml=result-unit.xml test\\unit
                     '''

                     junit 'result*.xml'

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
        
        stage('Coverage') {
            
            steps {
                
                bat '''
                    coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                    coverage xml
                '''
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,90'
                }
                
            }
            
        }
        
        
        stage('Codigo estatico') {
            
            steps {

                bat '''
                    flake8 --exit-zero --format=pylint app > flake8.out
                '''

                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            }
            
        }
        
        stage('Seguridad') {
            
            steps {
                
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''

                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                
            }
            
        }
        
        
        stage('Rendimiento') {
            
            steps {
                

                bat '''
                     C:\\Users\\Guille\\Desktop\\apache-jmeter-5.6.3\\bin\\jmeter -n -t CP1-2.jmx -f -l flask.jtl
                '''

                perfReport sourceDataFiles: 'flask.jtl'
                
            }
            
        }
        
    }
    
}