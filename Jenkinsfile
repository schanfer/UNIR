pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/schanfer/UNIR.git'
                bat "dir"
                bat "java -version"
            }
        }

         stage ('startFlask'){
            steps{
                bat '''
                    set FLASK_APP=app\\api.py
                    set FLASK_ENV=development
                    start /B flask run                '''
                script {
                    // Verificar si WireMock y Flask están corriendo (simplificado para la demostración)
                    waitUntil {
                        // Aquí podrías verificar si los puertos están escuchando, por ejemplo
                        def result = bat(script: 'netstat -an | findstr ":5000"', returnStatus: true)
                        return result == 0  // Si el puerto 5000 está en uso, 'findstr' devuelve 0
                    }
                }
            }
        }
        
         stage ('startRest'){
            steps{
                bat '''
                    start /B java -jar D:\\UNIR\\software\\wiremock\\wiremock-standalone-3.10.0.jar --port 9090 --root-dir D:\\UNIR\\software\\wiremock
                '''
                script {
                    // Verificar si WireMock y Flask están corriendo (simplificado para la demostración)
                    waitUntil {
                        // Aquí podrías verificar si los puertos están escuchando, por ejemplo
                        def result = bat(script: 'netstat -an | findstr ":9090"', returnStatus: true)
                        return result == 0  // Si el puerto 9090 está en uso, 'findstr' devuelve 0
                    }
                }
            }
        }
        stage('tests'){
            parallel{
                stage('Unit'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult:'FAILURE'){
                            bat '''
                            set PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                    } 
                }  
                stage('Rest'){
                    steps{
                        bat '''
                            SET PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-rest.xml test\\rest
                        '''
                    }
                }
                
            }
        }
        
        stage('Results'){
            steps{
                junit 'result*.xml'
            }
        }
    }
}
