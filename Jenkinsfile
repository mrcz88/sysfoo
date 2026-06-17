pipeline {
 agent any
 tools{
   maven 'Maven 3.9.6'
 }
 stages {
   stage("build"){
     steps{
       echo 'Compiling sysfoo app'
	   sh 'mvn compile'
    }
   }
   stage("two"){
     steps{
       echo 'step 2'
       sleep 9
     }
   }
   stage("three"){
     steps{
       echo 'step 3'
       sleep 5
     }
   }
 }
 post {
   always {
     echo 'This pipeline is completed..'
   }
 }
}
