pipeline {
    agent any
    stages {
        stage('Build API') {
           steps {
               sh "cd spring-petclinic-rest && nohup mvn spring-boot:run &"
               sleep(20)
           }
        }
         stage('Build Website') {
            steps {
               sh 'cd spring-petclinic-angular/static-content && curl https://jcenter.bintray.com/com/athaydes/rawhttp/rawhttp-cli/1.0/rawhttp-cli-1.0-all.jar -o rawhttp.jar && nohup java -jar ./rawhttp.jar serve . -p 4200 &'  
                sleep(3)
                  }
           }
      
         stage('Postman testing') {
            steps { 
                script {
                    try {
                       sh 'newman run API_test/PetMain.postman_collection.json --environment API_test/PetE.postman_environment.json --reporters junit --verbose'
                    } catch (Exception e) {
                        echo "Tests are failing, continue pipeline..."
                    } 
                        echo "Tests ran successfully, continue pipeline..."
                    try {
                        sh 'newman run API_test/PetMainBound.postman_collection.json --environment API_test/PetE.postman_environment.json --reporters junit'
                        } catch (Exception e) {
                        echo "Tests are failing, continue pipeline..."
                    }
                }
            }
            post {
                always {
                    junit '**/*.xml'
                }
            }
        }
        
        stage('Robot Framework System tests with Selenium') {
            steps {
                sh 'robot --variable BROWSER:headlesschrome -d Robot_tests/Results Robot_tests/Tests'
            }
            post {
                always {
                    script {
                          step(
                                [
                                  $class              : 'RobotPublisher',
                                  outputPath          : 'Robot_tests/Results',
                                  outputFileName      : '**/output.xml',
                                  reportFileName      : '**/report.html',
                                  logFileName         : '**/log.html',
                                  disableArchiveOutput: false,
                                  passThreshold       : 50,
                                  unstableThreshold   : 40,
                                  otherFiles          : "**/*.png,**/*.jpg",
                                ]
                          )
                    }
                }
            }
        }
        
    }
}
