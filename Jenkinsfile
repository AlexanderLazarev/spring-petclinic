pipeline
{
    agent
    {
        label 'Jenkins-Slave'
    }
    
    tools {
        maven 'Maven'
    }
    
    stages
    {
        stage('Build') 
        {
	    steps {
                git 'https://github.com/AlexanderLazarev/spring-petclinic.git'
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Test') 
        {
            input {
                message "Продолжить?"
                ok "Ok"
            }
	    steps {
                sh 'mvn test'
                script {
                    junit 'target/*-reports/*.xml, */target/*-reports/*.xml'
                }
            }
        }

        stage('Code analysis') 
        {
            input {
                message "Продолжить?"
                ok "Ok"
            }
		    steps {
			    script {
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv('SonarServer') 
                    {
                        sh "mvn sonar:sonar -Dsonar.projectKey=opendays "
                    }
                    sleep(10)
                    timeout(time: 1, unit: 'MINUTES')
                    {
                        withSonarQubeEnv('SonarServer')
                        {
                            def qp = waitForQualityGate()
                            print "Finished waiting"
                            if (qp.status != "OK")
                                error "Такой код нам не нужен!"
                        }
                    }
				}
			}
        }
        
        stage('Package to container') 
        {
            input {
                message "Продолжить?"
                ok "Ok"
            }
		    steps {
                sh 'docker build --build-arg JAR_FILE="target/spring-petclinic-2.1.0.BUILD-SNAPSHOT.jar" -t demo .'
			}
        }
        
        stage('Push to Docker Hub') 
        {
            input {
                message "Продолжить?"
                ok "Ok"
            }
		    steps {
                //sh 'docker push evsu/opendays:latest'
                sh 'docker push devopsepam/demo:v4'
			}
        }
        
        stage('Deploy') 
        {
            input {
                message "Продолжить?"
                ok "Ok"
            }
		    steps {
		        
		        script {
                    sh 'docker ps -aq | xargs --no-run-if-empty docker rm -f'
                    sh 'docker run -d -p 80:8080 demo'
                    print 'УСТАНОВКА ПРОШЛА УСПЕШНО!'
		        }
			}
        }
    }
}
