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
        stage("Publish over ssh"){
            steps{ 
                script{
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'Ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook /opt/k8s/create-simple-devops-image.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//k8s', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
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