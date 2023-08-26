pipeline {
  agent any 
  // tools {
  //   maven 'Maven'
  // }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
     }
    
    stage ('Check secrets') {
      steps {
      sh 'trufflehog3 https://github.com/roshangami/webgoat_devsecops.git -f json -o truffelhog_output.json || true'
      }
    }
    
    stage ('Generate build') {
      steps {
        sh 'mvn clean install -DskipTests'
      }
    }  
	  
   stage ('Deploy to server') {
            steps {
	   timeout(time: 3, unit: 'MINUTES') {
              sshagent(['app-server']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/webgoat-devsecops/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar security@13.233.109.110:/WebGoat'
		            sh 'ssh -o  StrictHostKeyChecking=no security@13.233.109.110 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=13.233.109.110 --server.port=9999 &"'
                }
	          }
        }     
    }
   
    stage ('DAST - OWASP ZAP') {
            steps {
           sshagent(['dast-server']) {
                sh 'ssh -o  StrictHostKeyChecking=no kundan@13.234.225.54 "sudo docker run --rm -v /home/apps:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://13.233.109.110:9999/WebGoat -x zap_report || true" '
              }      
           }       
    }
   }  
}
