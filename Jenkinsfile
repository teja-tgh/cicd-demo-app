pipeline {
    agent any

    environment {
        APPROVAL_EMAIL = 'teja.dannina@gmail.com'
    }

    stages {
        stage('Build Application') {
            steps {
                bat 'mvn clean install -DskipTests'
            }
        }
        stage('Testing') {
            steps {
                bat 'mvn clean test'
            }
        }
        

        stage('Publish to Exchange') {
            steps {
                script {
                    echo env.GIT_BRANCH?.toLowerCase() ?: 'branch not found'
                    bat 'mvn clean deploy'
                }
            }
        }

        stage('Approval') {
            steps {
                script {
                    sendApprovalEmail()

                    def approvalResponse = false

                    timeout(time: 5, unit: 'MINUTES') {
                        approvalResponse = input(
                            id: 'userInput',
                            message: 'Please approve the deployment to dev environment',
                            parameters: [
                                booleanParam(defaultValue: false, description: 'Approve Deployment?', name: 'Approve')
                            ]
                        )
                    }

                    if (!approvalResponse) {
                        error 'Deployment not approved. Aborting.'
                    }
                }
            }
        }

        stage('Build and Deploy') {
            steps {
                script {
                    deployToAnypoint('SandBox', 'cicd-demo-app')
                }
            }
        }
    }
	post {
	    success {
	        mail to: 'teja.dannina@gmail.com', 
	        subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
	        mimeType: 'text/html',
	        body: """
	             <html>
	                 <body>
	                     <h2 style="color: green;">Jenkins Build Successful!</h2>
	                     <p>Good news! The Jenkins build <strong>'${env.JOB_NAME}'</strong> #<strong>${env.BUILD_NUMBER}</strong> was successful.</p>
	                     <hr />
	                     <p><strong>Details:</strong></p>
	                     <ul>
	                         <li><strong>Job Name:</strong> ${env.JOB_NAME}</li>
	                         <li><strong>Build Number:</strong> ${env.BUILD_NUMBER}</li>
	                         <li><strong>Build Time:</strong> ${new Date()}</li>
	                         <li><strong>Triggered By:</strong> ${currentBuild.getBuildCauses()[0]?.userId ?: 'Automated Trigger'}</li>
	                         <li><strong>Branch:</strong> ${env.GIT_BRANCH ?: 'N/A'}</li>
	                         <li><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
	                     </ul>
	                     <p style="color: green;">Keep up the great work!</p>
	                 </body>
	             </html>
	             """
	    }
	    failure {
	        mail to: 'teja.dannina@gmail.com',
	             subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
	             mimeType: 'text/html',
	             body: """
	             <html>
	                 <body>
	                     <h2 style="color: red;">Jenkins Build Failed</h2>
	                     <p>Unfortunately, the Jenkins build <strong>'${env.JOB_NAME}'</strong> #<strong>${env.BUILD_NUMBER}</strong> failed.</p>
	                     <hr />
	                     <p><strong>Details:</strong></p>
	                     <ul>
	                         <li><strong>Job Name:</strong> ${env.JOB_NAME}</li>
	                         <li><strong>Build Number:</strong> ${env.BUILD_NUMBER}</li>
	                         <li><strong>Build Time:</strong> ${new Date()}</li>
	                         <li><strong>Triggered By:</strong> ${currentBuild.getBuildCauses()[0]?.userId ?: 'Automated Trigger'}</li>
	                         <li><strong>Branch:</strong> ${env.GIT_BRANCH ?: 'N/A'}</li>
	                         <li><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
	                     </ul>
	                     <p style="color: red;">Please review the logs and take necessary actions.</p>
	                 </body>
	             </html>
	             """
	    }
	}
	
}

def deployToAnypoint(environmentName,appName) {
    echo environmentName
    echo 'Deploying mule project due to the latest code commit…' 
    echo 'Deploying to the configured environment….' 
    bat """
        mvn clean deploy -DmuleDeploy \
        -Dmule_env=${environmentName} \
        -Dmule_appName=${appName}
    """
    }
def sendApprovalEmail() {
    def subject = "Approval Required: Jenkins Pipeline Deployment to Production Environment"
    def body = """
        <html>
        <body>
        <p>Dear Approver,</p>
        <p>A deployment to the Production environment is pending your approval.</p>
        <p>Please approve or deny using the Jenkins input step in the build page.</p>
        <p>Branch: ${env.GIT_BRANCH}</p>
        <p>View the build at: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
        <p>Thank you.</p>
        </body>
        </html>
    """
    mail to: "${APPROVAL_EMAIL}", subject: subject, body: body, mimeType: 'text/html'
}