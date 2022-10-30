pipeline{
    agent any 
    stages{
        stage("Github automation"){
            steps{
                git branch: 'master', url: 'https://github.com/mdsdsardar/k8s.git'
            }
        }
        stage("Maven Build"){
            tools {
                maven 'Maven' 
            }
            steps{    
                sh 'mvn clean install package'
            }
        }
        stage("SonarQube analysis"){
            tools {
                maven 'Maven'
            }
            steps{ 
                script{                    
                    withSonarQubeEnv(credentialsId: 'saadissad') {
                        sh 'mvn clean package sonar:sonar'

                    }
                        
                }   
                
            }
        }
        stage("Upload war file to nexus"){
            steps{
             
                script{
                    

                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo =  readPomVersion.version.endsWith("SNAPSHOT") ? "k8s-snapshots" : "k8s-release"
                    nexusArtifactUploader artifacts: 
                    [
                        [
                            artifactId: 'maven-project', classifier: '', 
                            file: "webapp/target/webapp.war", type: 'war'
                        ]
                    ],
                    credentialsId: 'nexus3', 
                    groupId: 'com.example.maven-project', 
                    nexusUrl: '13.234.38.99:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepo, 
                    version: "${readPomVersion.version}"     
                }               
            }   
                
        }

    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}