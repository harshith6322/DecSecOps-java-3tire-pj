pipeline{

    agent any
    // agent{
    //    label{
    //       label "slave-1"
    //       retries  3
    //    }

    // }

    tools {
      jdk 'jdk_tool'
      maven 'maven_tool'
    }

    environment{
      scannerHome= tool "sonar_tool"
    }


    stages{

        stage("Clean Workspace"){
            steps{
              cleanWs()
            }
        }

        stage("Code"){
            steps{
               git branch: 'main', url: 'https://github.com/harshith6322/DecSecOps-java-3tire-pj.git'

            }
        }

        stage("Compile Code"){
            steps{
              sh  "mvn clean install"

            }
        }

        stage("Unit Test"){
            steps{
                sh "mvn test"
            }
        }

        stage("CQA"){

            steps{
                script{
                    withSonarQubeEnv("sonar_server") {
                        sh """
                           ${scannerHome}/bin/sonar-scanner  \
                           -Dsonar.sources=src \
                           -Dsonar.java.binaries=target/classes \
                           -Dsonar.projectKey=java-3-tire     \
                            -Dsonar.exclusions=**/target/**,**/build/**,**/.git/**,**/.idea/**,**/.settings/**,**/out/** \
                        """

                    }
                }

            
            }
        }

        stage("Quality Gate"){
            steps{  
               
               waitForQualityGate abortPipeline: true, credentialsId: 'sonar_token'
               
            }
        }


        stage("OWASP Check"){
            steps{
                script{
                   dependencyCheck additionalArguments: '''--format HTML''', debug: true, odcInstallation: 'dpc_tool'
                }
            }
        }


    }

}