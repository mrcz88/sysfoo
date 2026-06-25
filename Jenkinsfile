/* non e' una scripted pipeline. La scripted pipeline comincia con node {
   ed è codificata in Groovy
*/

/* DECLARATIVE PIPELINE */
pipeline {
 agent any
 /* parameter */
 parameters {
   booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip Maven tests')
 }
  
 environment {
  APP_NAME = 'my-java-app'
  MAVEN_OPTS = '-Xmx512m'
 }
	
 tools{
   maven 'Maven 3.9.6'
 }

 /* all stages of the pipeline */
 stages {
   stage("build"){
     steps{
       echo 'Compiling sysfoo app'
	   sh 'mvn compile'
    }
   }
	stage('Test') {
		when {
			expression { return !params.SKIP_TESTS }
		}
		steps {
			timeout(time: 5, unit: 'MINUTES') {
				retry(2) {
					sh 'mvn test'
				}
			}
		}
	}
	stage('Package') {
		steps {
			sh 'mvn package -DskipTests'
			archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
		}
	}
 }
 post {
	always {
		echo 'Pipeline finished'
		junit 'target/surefire-reports/*.xml'
	}
	success {
		echo 'Build successful'
	}
	failure {
		echo 'Build failed'
	}
 }
}
