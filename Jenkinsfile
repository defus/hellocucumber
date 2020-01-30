def pom, isMultiModuleBuild, pomModule
pipeline {
    agent none

    stages {
        stage('Checkout') {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
                checkout([$class: 'GitSCM'])
            }
        }

        // BUILD START
        stage('Build') {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
                // Run the maven build
                script {
                    pom = readMavenPom file: 'pom.xml'
                    sh "mvn clean install"
                }
            }
        }

        stage('Generate Cucumber HTML report') {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
                cucumber buildStatus: 'UNSTABLE',
                    fileIncludePattern: '**/*.json',
                    trendsLimit: 10,
                    classifications: [
                        [
                            'key': 'Browser',
                            'value': 'Firefox'
                        ]
                    ]
            }
        }

        stage("Dependency Check") {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
              script{
                  isMultiModuleBuild = pom.modules.size() != 0
                    if(isMultiModuleBuild) {
                      println "Maven Multimodule Build, Modules: ${pom.modules}"
                      pom.modules.each {
                        dir("$it") {
                          println "Multimodule Build, start Sonarqube Analysis for Module $it."
                          sh("mkdir -p build/owasp")
                          dependencycheck additionalArguments: '--project plastinforme --scan ./ --data /data/jenkins/security/owasp-nvd/ --out build/owasp/dependency-check-report.xml --format XML --proxyserver localhost --proxyport 5865', odcInstallation: 'dependency-check'
                          dependencyCheckPublisher pattern: 'build/owasp/dependency-check-report.xml'
                        }
                      }
                    }else{
                          sh("mkdir -p build/owasp")
                          dependencycheck additionalArguments: '--project plastinforme --scan ./ --data /data/jenkins/security/owasp-nvd/ --out build/owasp/dependency-check-report.xml --format XML --proxyserver localhost --proxyport 5865', odcInstallation: 'dependency-check'
                          dependencyCheckPublisher pattern: 'build/owasp/dependency-check-report.xml'


                    }

              }
            }
        }


        stage ("Dependency Track"){
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
               script{

                  if(isMultiModuleBuild) {
                      println "Maven Multimodule Build, Modules: ${pom.modules}"
                      pom.modules.each {
                        dir("$it") {
                            pomModule = readMavenPom file: 'pom.xml'
                            artifactId = pomModule.artifactId
                            version = pomModule.version

                            
                            if ( !artifactId?.trim() )
                                artifactId = pomModule.parent.artifactId

                            
                            if ( !version?.trim() )
                                version = pomModule.parent.version
                          sh("mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom")

                         dependencyTrackPublisher artifact: 'target/bom.xml', artifactType: 'bom', projectName: "${artifactId}", projectVersion: "${version}", synchronous: true
                        }
                      }
                  }else{
                          sh("mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom")

                         dependencyTrackPublisher artifact: 'target/bom.xml', artifactType: 'bom', projectName: "${pom.artifactId}", projectVersion: "${pom.version}", synchronous: true
                  }
               }

             }
         }


    }
}

def Properties getProperties(filename) {
    def properties = new Properties()
    properties.load(new StringReader(readFile(filename)))
    return properties
}

@NonCPS
def jsonParse(text) {
    return new groovy.json.JsonSlurperClassic().parseText(text);
}

@NonCPS
def jsonParse(URL url, String basicAuth) {
    def conn = url.openConnection()
    conn.setRequestProperty( "Authorization", "Basic " + basicAuth )
    InputStream is = conn.getInputStream();
    def json = new groovy.json.JsonSlurperClassic().parseText(is.text);
    conn.disconnect();
    return json
}
