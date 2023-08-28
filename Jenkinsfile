pipeline {
	agent any

	stages {
		stage ('Initialize') {
		      steps {
			sh '''      
		        	echo "PATH = ${PATH}"
	   			echo "M2_HOME = ${M2_HOME}"
       			'''
      			}
    		}
            stage ('Check Secrets') {
     			steps {
 			    sh 'trufflehog3 https://github.com/Kundanpagare/infra-SecOps.git -f json -o truffelhog_output.json || true'
 			      
			}
    		}
            stage('Compile and Build'){
 		steps{
 			sh 'mvn clean install -DskipTests'
			}
 		}
	//	stage ('Fetch Application server') {
   	//	     steps {
 	//		     sshagent(['ubuntu-server']) {
 	//		sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/pipeline_webgoat_devsecops/target/webgoatserver-v8.2.0.jar ubuntu@3.7.69.183:~/WebGoat'
	//			 }
	//		       }
        //                }
 	stage ('Deploy to server') {
            steps {
 		    timeout(time: 4, unit: 'MINUTES') {
	        	   sshagent(['app-key']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/jenkins/target/webgoat-2023.5-SNAPSHOT.jar app@35.154.75.51:WebGoat'
		sh 'ssh -o  StrictHostKeyChecking=no app@35.154.75.51 "nohup java -jar /WebGoat/webgoat-2023.5-SNAPSHOT.jar --server.address=13.233.223.161 --server.port=9090 &"'
 				
           		}
		    }
            	}     
    	}
	
 	 stage ('Dynamic analysis') {
            steps {
          	  sshagent(['dast-key']) {
            sh 'ssh -o  StrictHostKeyChecking=no ubuntu@43.205.239.13 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://35.154.75.51/WebGoat -x zap_report || true" '
			
                  }      
            }       
    }
    }
}
