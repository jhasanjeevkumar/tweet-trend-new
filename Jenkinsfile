def registry = 'https://sjha01.jfrog.io'
def imageName = 'https://sjha01.jfrog.io/sjha01-docker-docker-local/ttrend'
def version = '2.1.2'
pipeline {
    agent {
        node{
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    }

    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('sonar-analysis') {
            environment {
                scannerHome = tool 'sjha01-sonar-scanner'
            }
            steps{
                withSonarQubeEnv('sjha01-sonarqube-server') {
                    sh 'touch sonar-analysis.txt'
                }
            }

        }
        stage("Jar Publish") {
            steps {
                script {
                        echo '<----1----------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifactory-cred"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "jarstaging/(*)",
                                  "target": "libs-release-local/{1}",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<-----1---------- Jar Publish Ended --------------->'  
                
                }
            }   
        }
        stage(" Docker Build ") {
          steps {
            script {
               echo '<--------------- Docker Build Started --------------->'
               app = docker.build(imageName+":"+version)
               echo '<--------------- Docker Build Ends --------------->'
            }
          }
        }

        stage (" Docker Publish ") {
            steps {
                script {
                   echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'artifactory-cred'){
                        app.push()
                    }    
                   echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }   
    }
}
