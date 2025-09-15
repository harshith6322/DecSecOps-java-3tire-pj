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
       USERNAME_DOCKER="harshithreddy6322"
       IMAGE_TAG="${BUILD_ID}"
       IMAGE_NAME="appimage"
       IMAGE_NAME2="dbimage"
       RECIPIENTS="harshithreddy6322@gmail.com,noreply.jenkins2025@gmail.com"
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

        stage("Build"){
            steps{
              sh  "mvn clean install"
              sh "cp -r target Docker-app"

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

                dependencyCheck additionalArguments: ''' --scan target/  --out dependency-check-report ''', debug: true, nvdCredentialsId: 'owasp_token', odcInstallation: 'dpc_tool'
                dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'

              }
                
                
            }
        }

        stage("Nexsus"){
            steps{

                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexus_token', groupId: 'com.visualpathit', nexusUrl: '13.222.125.210:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'java-3-tire', version: 'v2'

            }
        }


        stage("Build Images"){
            steps{

                sh "docker build -t ${IMAGE_NAME} Docker-app"
                sh "docker build -t ${IMAGE_NAME2} Docker-db"
               
            }

        }

        stage("Trivy Scan"){
            steps{

              sh "trivy image ${IMAGE_NAME} > ${IMAGE_NAME}_scan_report.txt"
              sh "trivy image ${IMAGE_NAME2} > ${IMAGE_NAME2}_scan_report.txt"
            }

           
            
        }

        stage('Dcoker Compose') {
            steps {
              sh "docker-compose up -d"
              sh "sleep 3m"
              sh "docker-compose down"
                   
            }
        }


    }


     post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: getEmailBody("SUCCESS"),
                mimeType: 'text/html',
                to: "${RECIPIENTS}",
                attachLog: true,
                compressLog: true,
                attachmentsPattern: 'npm-audit-report.json,trivy_scan_ouput.txt' 
               
            )
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: getEmailBody("FAILURE"),
                mimeType: 'text/html',
                to: "${RECIPIENTS}",
                attachLog: true,
                compressLog: true,
                attachmentsPattern: 'npm-audit-report.json,trivy_scan_ouput.txt' 
                
                
            )
        }

        aborted{
            emailext(
                subject: "⚠️ Aborted: ${JOB_NAME} #${BUILD_NUMBER}",
                body: getEmailBody("FAILURE"),
                mimeType: 'text/html',
                to: "${RECIPIENTS}",
                attachLog: true,
                compressLog: true,
                attachmentsPattern: "${IMAGE_NAME}_scan_report.txt,${IMAGE_NAME2}_scan_report.txt,"
                
                
            )


        }
    }
}
// Function to load HTML email body
@NonCPS
def getEmailBody(status) {
    return """
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; background:#f6f9fc; padding:20px; }
        .container { max-width:600px; margin:auto; background:#fff; border-radius:8px;
                     box-shadow:0 2px 8px rgba(0,0,0,0.1); padding:20px; }
        h2 { text-align:center; }
        .status-success { color: #2e7d32; font-weight: bold; }
        .status-failed { color: #c62828; font-weight: bold; }
        .button { display:inline-block; padding:10px 20px; margin-top:10px; 
                  font-size:14px; font-weight:bold; text-decoration:none;
                  border-radius:4px; color:#fff; }
        .btn-success { background:#2e7d32; }
        .btn-failed { background:#c62828; }
        .footer { text-align:center; font-size:12px; color:#777; margin-top:20px; }
      </style>
    </head>
    <body>
      <div class="container">
        <h2>CI/CD Pipeline Report</h2>
        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
        <p><strong>Branch:</strong> ${env.BRANCH_NAME ?: 'N/A'}</p>
        <p><strong>Build Number:</strong> #${env.BUILD_NUMBER}</p>
        <p><strong>Status:</strong> <span class="${status == 'SUCCESS' ? 'status-success' : 'status-failed'}">${status}</span></p>
        <a class="button ${status == 'SUCCESS' ? 'btn-success' : 'btn-failed'}" href="${env.BUILD_URL}">View Build Logs</a>
        <div class="footer">
          <p>This is an automated message from Jenkins CI/CD</p>
          
        </div>
      </div>
    </body>
    </html>
    """
} 

