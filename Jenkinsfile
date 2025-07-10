node{
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
		ssh -o StrictHostKeyChecking=no ec2-user@52.66.81.148 'sudo systemctl stop tomcat'
		sleep 10
		"""
	  }
	}
    stage("Deploy war to tomcat"){
        sshagent(['Tomcat_server']) {
    sh "scp -o strictHostKeyChecking=no target/student-reg-webapp.war ec2-user@52.66.81.148:/opt/tomcat/webapps/"
        }
    }
    
	stage("start tomcat service"){
	  sshagent(['Tomcat_server']){
	    sh "ssh -o StrictHostKeyChecking=no ec2-user@52.66.81.148 'sudo systemctl start tomcat'"

	  }
	}
}
	catch (err){
		echo "An error occured: ${e,getMessage()}"
        currentBuild.result = 'FAILURE'
	}
    finally{
        def buildStatus = currentBuild.result ?: 'SUCCESS'
        sendEmail(
            subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build ${buildStatus}"
            body: "Build ${buildStatus}.Please check the console output at ${env.BUILD_URL}"
            recipient: 'arathisk12@gmail.com'
        )
    }
}

def sendEmail(String subject, String body, String recipient)
{
    emailext(
        subject: subject,
        body: body,
        to: recipient,
        mimeType: 'text/html'
    )
}
