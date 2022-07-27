pipeline {                                                                           
  agent any
  tools {
        maven "mvn3.8.6"
  }
  environment {
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "http"
    NEXUS_URL = "192.168.1.24:8081"
    NEXUS_REPOSITORY = "vprofile-repo"
    NEXUS_REPO_ID    = "vprofile-repo"
    NEXUS_CREDENTIAL_ID = "nexuslogin"
    ARTVERSION = "${env.BUILD_ID}"
  }

  stages{                                                                            
    stage('Fetch Code'){                                                             
      steps{                                                                         
        git branch: 'main', url:'https://github.com/darkrune0/vprofile-PaaC.git'
      }                                                                              
    }                                                                                
    stage('build'){                                                                  
      steps{                                                                         
        sh 'mvn clean install -DskipTests'                                                             
      }
      post{
        success{
            echo "Now archiving."
            //archiveArtifacts artifacts: '**/*.war'
        }
      }                                                                      
    }                                                                                
    stage('Test'){                                                                   
      steps{                                                                         
        sh 'mvn test'                                                                
      }                                                                              
    }
    stage('checkstyle Analysis'){
        steps{
            sh 'mvn checkstyle:checkstyle'
        }
    }
    //this stage calls for a sonarqube report
    stage('Sonar Analysis'){
        //env variable
        environment{
            scannerHome = tool 'sonar4.7'
        }
        steps{
            withSonarQubeEnv('sonar'){
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                  
                '''
            }
        }
    }
    //this step makes jenkins wait for sonarqube report
    stage('Quality Gate 666'){
        steps{
            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
        }
    }
    stage('upload artifact'){
        steps{
            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol:'http',
                nexusUrl: '192.168.1.24:8081',
                groupId: 'QA',
                version: "${env.BUILD_ID}",
                repository:'vprofile-repo',
                credentialsId:'nexuslogin',
                artifacts: [
                    [artifactID: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                ]
                )

        }
    }
    stage("Publish to Nexus Repository Manager") {
        steps {
            script {
                pom = readMavenPom file: "pom.xml";
                filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;

                echo NEXUS_VERSION;
                echo NEXUS_CREDENTIAL_ID;
                if(artifactExists) {
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: ARTVERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: "pom.xml",
                            type: "pom"]
                        ]
                    );
                } 
                else {
                    error "*** File: ${artifactPath}, could not be found";
                }
            }
        }
    }


  }                                                                                  
}                                                                                    
// ##########################################################################
// pipeline {
    
// 	agent any
	
// 	tools {
//         maven "maven3"
//     }

//     environment {
//         NEXUS_VERSION = "nexus3"
//         NEXUS_PROTOCOL = "http"
//         NEXUS_URL = "192.168.1.24:8081"
//         NEXUS_REPOSITORY = "vprofile-release"
// 	       NEXUS_REPO_ID    = "vprofile-release"
//         NEXUS_CREDENTIAL_ID = "nexuslogin"
//         ARTVERSION = "${env.BUILD_ID}"
//     }
	
//     stages{
        
//         stage('BUILD'){
//             steps {
//                 sh 'mvn clean install -DskipTests'
//             }
//             post {
//                 success {
//                     echo 'Now Archiving...'
//                     archiveArtifacts artifacts: '**/target/*.war'
//                }
//             }
//         }

// 	stage('UNIT TEST'){
//             steps {
//                 sh 'mvn test'
//             }
//         }

// 	stage('INTEGRATION TEST'){
//             steps {
//                 sh 'mvn verify -DskipUnitTests'
//             }
//         }
		
//         stage ('CODE ANALYSIS WITH CHECKSTYLE'){
//             steps {
//                 sh 'mvn checkstyle:checkstyle'
//             }
//             post {
//                 success {
//                     echo 'Generated Analysis Result'
//                 }
//             }
//         }

//         stage('CODE ANALYSIS with SONARQUBE') {
          
// 		  environment {
//              scannerHome = tool 'sonarscanner4'
//           }

//           steps {
//             withSonarQubeEnv('sonar-pro') {
//                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
//                    -Dsonar.projectName=vprofile-repo \
//                    -Dsonar.projectVersion=1.0 \
//                    -Dsonar.sources=src/ \
//                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
//                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
//                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
//                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
//             }

//             timeout(time: 10, unit: 'MINUTES') {
//                waitForQualityGate abortPipeline: true
//             }
//           }
//         }

//         stage("Publish to Nexus Repository Manager") {
//             steps {
//                 script {
//                     pom = readMavenPom file: "pom.xml";
//                     filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
//                     echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
//                     artifactPath = filesByGlob[0].path;
//                     artifactExists = fileExists artifactPath;
//                     if(artifactExists) {
//                         echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
//                         nexusArtifactUploader(
//                             nexusVersion: NEXUS_VERSION,
//                             protocol: NEXUS_PROTOCOL,
//                             nexusUrl: NEXUS_URL,
//                             groupId: pom.groupId,
//                             version: ARTVERSION,
//                             repository: NEXUS_REPOSITORY,
//                             credentialsId: NEXUS_CREDENTIAL_ID,
//                             artifacts: [
//                                 [artifactId: pom.artifactId,
//                                 classifier: '',
//                                 file: artifactPath,
//                                 type: pom.packaging],
//                                 [artifactId: pom.artifactId,
//                                 classifier: '',
//                                 file: "pom.xml",
//                                 type: "pom"]
//                             ]
//                         );
//                     } 
// 		    else {
//                         error "*** File: ${artifactPath}, could not be found";
//                     }
//                 }
//             }
//         }
//     }
// }
