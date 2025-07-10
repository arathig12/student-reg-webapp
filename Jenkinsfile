node{
	def buildStatus = 'SUCCESS'  // Default status

	try{
		
    def mavenHome =tool name: 'Maven-3.9.10', type: 'maven'
    stage("Git Clone")
    {
        echo "Git Clone"
        git branch: 'development', credentialsId: 'GitHubCred', url: 'https://github.com/arathig12/student-reg-webapp.git'
    }
    stage("Package")
    {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage("verify"){
        withCredentials([string(credentialsId: 'sonarToken', variable: 'sonarToken')]) {
        sh "${mavenHome}/bin/mvn clean verify sonar:sonar -Dsonar.token=${sonarToken}"
        }
    }
    stage("Deploy"){
         sh "${mavenHome}/bin/mvn clean deploy"
    }
	
	stage("stop tomcat service"){
	  sshagent(['Tomcat_server']){
        sh """	    
		ssh -o StrictHostKeyChecking=no ec2-user@65.0.27.90 'sudo systemctl stop tomcat'
		sleep 10
		"""
	  }
	}
    stage("Deploy war to tomcat"){
        sshagent(['Tomcat_server']) {
    sh "scp -o strictHostKeyChecking=no target/student-reg-webapp.war ec2-user@65.0.27.90:/opt/tomcat/webapps/"
        }
    }
    
	stage("start tomcat service"){
	  sshagent(['Tomcat_server']){
	    sh "ssh -o StrictHostKeyChecking=no ec2-user@65.0.27.90 'sudo systemctl start tomcat'"

	  }
	}
}
	catch (err){
		echo "An error occured: ${err.getMessage()}"
        currentBuild.result = 'FAILURE'
		buildStatus = 'FAILURE' 
		throw err
	}
finally {
    script {
        def buildStatus = currentBuild.result ?: 'SUCCESS'

        // Set color: green for SUCCESS, red for FAILURE/others
        def color = buildStatus == 'SUCCESS' ? 'green' : 'red'

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

        emailext(
            subject: subject,
            body: body,
            mimeType: 'text/html',
            to: 'arathisk12@gmail.com'
        )
    }
}


}	
  
