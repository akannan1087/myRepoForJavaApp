pipeline {
    agent any

    stages {
        
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
        }
        
        stage ("Code scan") {
            steps {
             withSonarQubeEnv("My_SonarQube") {
                sh "mvn sonar:sonar -f MyWebApp/pom.xml"
             }   
            }
        }
        
        stage ("code coverage") {
            steps {
                jacoco()
            }
        }
        
        stage ("Binary Upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: 'c63a1a05-a222-46fe-963f-7c3795beef41', groupId: 'com.dept.app', nexusUrl: 'ec2-18-212-254-182.compute-1.amazonaws.com:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ("DEV Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'e8ecbf78-f160-4fb9-8ec6-ab021f5440ed', path: '', url: 'http://ec2-3-91-190-139.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("Slack") {
            steps {
                slackSend channel: 'mar-2023-weekday-batch', message: 'DEV Deployment was success, please start your smoke testing..'
            }
        }
        
      //CD starts here
  stage ("DEV approve") {
    steps {
    echo "Taking approval from DEV Manager for QA Deployment"     
    timeout(time: 1, unit: 'HOURS') {
    input message: 'Do you approve QA Deployment?', submitter: 'admin,dev_manager@email.com,mysecon'
    }
  }
  }
    stage ("QA deploy") {
      steps {
        deploy adapters: [tomcat9(credentialsId: 'e8ecbf78-f160-4fb9-8ec6-ab021f5440ed', path: '', url: 'http://ec2-3-91-190-139.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
      }
     }
  
    stage ("QA notify") {
          steps {
            slackSend channel: 'mar-2023-weekday-batch', message: 'QA Deployment is done, please start testing..'
          }
         }
  
      stage ("QA Approve") {
          steps {
          echo "Taking approval from QA Manager for PROD Deployment"     
        }
      }
  
      stage ("PROD deploy") {
          steps {
            deploy adapters: [tomcat9(credentialsId: 'e8ecbf78-f160-4fb9-8ec6-ab021f5440ed', path: '', url: 'http://ec2-3-91-190-139.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
          }
       }
  
      stage ("Final notify") {
          steps {
          slackSend channel: 'mar-2023-weekday-batch, my-devops-team-channel, product-owners-teams', message: 'PROD Deployment is done, please start testing..'
        }
      }
    }
}
