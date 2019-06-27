pipeline
{
  agent { label 'JenkinsNode3||JenkinsNode4' }
  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(artifactDaysToKeepStr: '7', artifactNumToKeepStr: '5', daysToKeepStr: '7', numToKeepStr: '5'))
    //timestamps()
  }
  triggers {
    githubPush()
  }

  environment
  {
    // global server level
    DEPLOY_SERVER = 'ucdeploy1'
    UCD_SERVER = 'http://ucdeploy1'
    DEPLOY_REQPROCESS = "${UCD_SERVER}" + '/cli/applicationProcessRequest/request'
    NEXUS_PROTO = "http"
    NEXUS_HOST = "nexusrepo1"
    NEXUS_PORT = "8081"

    // job specific
    GIT_REPO = 'http://thpgithub/ASE/THPDevopsDemo1.git'
    NEXUS_CREDSID = 'NEXUS'
    NEXUS_REPOSITORY = 'gitdemo'
    NEXUS_GROUP = ''
    DEPLOY_ENV_TARGET = 'DEV'
    DEPLOY_APP_NAME = 'JPetStore'
    DEPLOY_APP_PROCESS = 'Deploy'
    DEPLOY_COMP_NAME = 'JPetStore-app'

  }
 
 tools {
      gradle 'Gradle2-3'
      jdk 'Node_JDK_Location'
  }
 
  parameters {
    booleanParam (
      name: 'AUTO_DEPLOY',
      defaultValue: true,
      description: 'Post-build deployment by default'
    )
  }

  stages
  {
  
    stage( "Checkout Source"){
	    
      steps{
	cleanWs()     
        checkout scm      
      }
    }

    stage( "Set up Environment Variables" ) {
      steps{
        script {

          def props = readProperties file: 'gradle.properties'
	      def version = props['version']
        
          APP_ID = props['name']
          NEXUS_GROUP = props['group']

          // expecting timestamp to be in yyyyMMdd-HHmmss format
          VERSION = "${version}_${BUILD_TIMESTAMP}"
          VERSION_TAG="${VERSION}"
          ARTIFACT_FILENAME="${APP_ID}-${version}.war"
          currentBuild.displayName = "${VERSION_TAG}"
        }
        sh "echo 'version: ${VERSION}'"
        sh "echo 'version_tag: ${VERSION_TAG}'"
        sh "echo 'articat_filename: ${ARTIFACT_FILENAME}'"
      }
    }

    stage('Build, Test, and Package') {
      steps
      {
        sh 'gradle -g /tmp war'
      }
    } 
	  
    //stage('Git Tag') {
       //steps
       //{
	 //withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'thpgithub-Login', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
		 //sh("git tag -a ${VERSION} -m 'Jenkins'")
	    //sh("git push http://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@thpgithub/ASE/THPDevopsDemo1.git --tags")
         //}
	//}	
     //}
	
    //stage('Publish to Nexus') {
      //steps
      //{
        //nexusArtifactUploader artifacts:
          //[[artifactId: APP_ID, classifier: '', file: "build/libs/${ARTIFACT_FILENAME}", type: 'war']],
          //credentialsId: NEXUS_CREDSID,
          //groupId: NEXUS_GROUP,
          //nexusUrl: "$NEXUS_HOST:$NEXUS_PORT",
          //nexusVersion: 'nexus3',
          //protocol: NEXUS_PROTO,
          //repository: NEXUS_REPOSITORY,
          //version: VERSION
      //}
    //}
	
  stage ('Approval'){
	steps{
		
	input message: 'Approve or Abort', parameters: [choice(choices: ['Approve', 'Abort'], description: '', name: 'Input')], submitter: 'nicole_Rosania@tufts-health.com', submitterParameter: 'submitter'
		//emailext (
		//		subject: "Jenkins Build '${currentBuild.fullDisplayName}: Job ${JOB_NAME}'",
		//		body: "'<a href=${BUILD_URL}input'>click to approve</a>",  
		//		recipientProviders: [[$class: 'nicole_rosania@tufts-health.com'], [$class: 'RequesterRecipientProvider']])
		  //  def userInput = inputId: 'userInput',
			//				message: 'Approve?',
			//				submitterParameter: 'submitter',
			//				submitter: 'nicole',
			//				parameters: [
			//				[$class: 'TextParameterDefinition', defaultValue: 'dev', description: 'Environment', name: 'env']]}
									
			//echo "Env: "+ userInput['env]
			//echo "submitted: "+ userInput['submitter']
	}}
	

    stage('Push to UrbanCode Deploy') {
      when { expression{ return params.AUTO_DEPLOY } }
      steps
      {
        step([$class: 'UCDeployPublisher',
          siteName: DEPLOY_SERVER,
          component: [
              $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
              componentName: DEPLOY_COMP_NAME,
              delivery: [
                  $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
                  pushVersion: VERSION,
                  baseDir: "${WORKSPACE}/build/libs",
                  fileIncludePatterns: '*.war',
                  fileExcludePatterns: '',
                  pushProperties: 'jenkins.server=Local\njenkins.reviewed=false',
                  pushDescription: "Pushed from Jenkins Pipeline build ${BUILD_ID}",
                  pushIncremental: false
                  ]
              ]
        ])
      }
    }
	
    stage('Deploy to DEV') {
      when { expression{ return params.AUTO_DEPLOY } }
      steps{
                sh 'rm -f applicationDeploy.json'
                sh 'touch applicationDeploy.json'
                sh 'echo "{" >> applicationDeploy.json'
                sh 'echo "\\"application\\" : \\"JPetStore\\"," >> applicationDeploy.json'
                sh 'echo "\\"applicationProcess\\": \\"Deploy\\","  >> applicationDeploy.json'
                sh 'echo "\\"environment\\": \\"DEV\\"," >> applicationDeploy.json'
                sh 'echo "\\"onlyChanged\\": \\"false\\"," >> applicationDeploy.json'
                sh 'echo "\\"versions\\": [{" >> applicationDeploy.json' 
                
                //deploy JPetStore-app
                sh "echo '\"version\": \"${VERSION}\",' >> applicationDeploy.json"
                sh 'echo "\\"component\\": \\"JPetStore-app\\"" >> applicationDeploy.json'
                sh 'echo "}," >> applicationDeploy.json'
                
                //deploy JPetstore-DB
                sh 'echo "{" >> applicationDeploy.json'
                sh 'echo "\\"version\\": \\"1.1\\"," >> applicationDeploy.json'
                sh 'echo "\\"component\\": \\"JPetStore-DB\\"" >> applicationDeploy.json'
                sh 'echo "}," >> applicationDeploy.json'
                
                //deploy JPetstore-WEB
                sh 'echo "{" >> applicationDeploy.json'
                sh 'echo "\\"version\\": \\"1.1\\"," >> applicationDeploy.json'
                sh 'echo "\\"component\\": \\"JPetStore-WEB\\"" >> applicationDeploy.json'
                
                sh 'echo "}]" >> applicationDeploy.json'
                sh 'echo "}" >> applicationDeploy.json'
            }//End steps
        }//End setup deployment process
        
    stage('Deploy Application'){
        steps{
                withCredentials([usernameColonPassword(credentialsId: 'UCD-Login', variable: 'UCDLOGIN')]) {
                    sh 'curl --fail -k -u $UCDLOGIN $DEPLOY_REQPROCESS -X PUT -d @applicationDeploy.json'
                }//End with credentials
        }
    }	
   //stage('Update JIRA') {
       //steps{
	    //withEnv(['JIRA_SITE=JIRAPROD']) {
		//jiraAddComment idOrKey: 'AD-553', comment: 'This build and artifact promotion was successful. The deployment process has been started in UrbanCode Deploy.'
	     //}
	//}	
    //}
  }
	
post {
      always {
	      emailext (
		subject: "Jenkins Build '${currentBuild.currentResult}: Job ${JOB_NAME}'",
		body: "'${currentBuild.currentResult}: Job ${JOB_NAME} build ${BUILD_NUMBER}'\n Check console output at ${BUILD_URL} to view the results.",  
		recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']])
			
	}
  }	
}
