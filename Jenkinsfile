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

        stage ('Build') {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
                script {
                    pom = readMavenPom file: 'pom.xml'
                    powershell "mvn clean install -Psimple-tests"
                }
            }
        }

        stage('Generate Cucumber HTML report') {
            agent { label 'windows-tests-fonctionnels-1' }
            steps {
                script {
                    try {
                        pom = readMavenPom file: 'pom.xml'
                        powershell "mvn clean install -Pcucumber-tests"

                        cucumber buildStatus: 'UNSTABLE',
                            fileIncludePattern: '**/*.json',
                            trendsLimit: 10,
                            classifications: [
                                [
                                    'key': 'Browser',
                                    'value': 'Chrome'
                                ]
                            ]
                    } catch (Exception e) {
                        cucumber buildStatus: 'UNSTABLE',
                            fileIncludePattern: '**/*.json',
                            trendsLimit: 10,
                            classifications: [
                                [
                                    'key': 'Browser',
                                    'value': 'Chrome'
                                ]
                            ]
                        
                        error("Le build a échoué à cause de cucumber")
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
