pipeline {
    agent any
    tools
    {
    maven 'maven3'
    }
  
    stages{
      stage ('Initialize'){
        steps{
          sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
             '''   
        }
      }
        
       stage ('git-secrets-check'){
           steps{
               sh 'rm trufflehog || true'
               sh 'docker run gesellix/trufflehog --json https://github.com/ravisinghrajput95/DevSecOps-pipeline.git > trufflehog'
               sh 'cat trufflehog'
               
           }
         }
        
        stage ('Source-Composition-Analysis'){
            steps{
                
                sh 'rm owasp* || true'
                sh 'wget "https://raw.githubusercontent.com/ravisinghrajput95/DevSecOps-pipeline/main/owasp-dependency-check.sh" '
                sh 'chmod +x owasp-dependency-check.sh'
                sh 'bash owasp-dependency-check.sh'
                sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
            }
        }
        
        stage ('SAST'){
            steps {
                withSonarQubeEnv('sonar') {
                   sh 'mvn sonar:sonar'
                   sh 'cat target/sonar/report-task.txt'
           }
          }
        }
            
      
      stage ('Build'){
        steps {
          sh 'mvn clean package'
          
        }
       }
        
        stage ('Deploy-to-Tomcat'){
        steps {
            sshagent(['tomat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war root@10.182.0.2:/var/lib/tomcat8/webapps/webapp.war'
        }
        }
      }
        
        stage ('DAST'){
          steps {
            sshagent(['zap']) {
              sh 'ssh -o  StrictHostKeyChecking=no root@10.182.0.3 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://34.125.14.173/:8080/webapp/" || true'
           }
         }
       }
    }
}
