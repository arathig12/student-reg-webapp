node {
    def buildStatus = 'SUCCESS'  // Default status

    try {
        def mavenHome = tool name: 'Maven-3.9.10', type: 'maven'

        stage("Git Clone") {
            echo "Git Clone
            git branch: 'development', credentialsId: 'GitHubCred', url: 'https://github.com/arathig12/student-reg-webapp.git'
        }

        stage("Package") {
            sh "${mavenHome}/bin/mvn clean package"
        }

        stage("Verify") {
            withCredentials([string(credentialsId: 'sonarToken', variable: 'sonarToken')]) {
                sh "${mavenHome}/bin/mvn clean verify sonar:sonar -Dsonar.token=${sonarToken}"
            }
        }

        stage("Deploy") {
            sh "${mavenHome}/bin/mvn clean deploy"
        }

        stage("Stop Tomcat") {
            sshagent(['Tomcat_server']) {
                sh "ssh -o StrictHostKeyChecking=no ec2-user@65.0.27.90 'sudo systemctl stop tomcat'"
            }
        }

        stage("Deploy WAR to Tomcat") {
            sshagent(['Tomcat_server']) {
                sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@65.0.27.90:/opt/tomcat/webapps/"
            }
        }

        stage("Start Tomcat") {
            sshagent(['Tomcat_server']) {
                sh "ssh -o StrictHostKeyChecking=no ec2-user@65.0.27.90 'sudo systemctl start tomcat'"
            }
        }

    } catch (err) {
        echo "💥 An error occurred: ${err.getMessage()}"
        buildStatus = 'FAILURE'
        currentBuild.result = 'FAILURE'
    } finally {
        // Send email in all cases
        def color = (buildStatus == 'SUCCESS') ? 'green' : 'red'
        def subject = "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${buildStatus}"
        def body = """
            <html>
            <body style="font-family: Arial, sans-serif; font-size: 14px;">
                <p><strong>Job Name:</strong> ${env.JOB_NAME}</p>
                <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                <p><strong>Build Status:</strong> <span style="color:${color}; font-weight:bold;">${buildStatus}</span></p>
                <p><strong>Console Output:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            </body>
            </html>
        """

        // ✅ Always inside script block
        script {
            emailext(
                subject: subject,
                body: body,
                mimeType: 'text/html',
                to: 'arathisk12@gmail.com'
            )
        }
    }
}
