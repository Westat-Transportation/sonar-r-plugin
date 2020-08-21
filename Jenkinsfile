def sonar_host = "sonarqube.westat.com"
def plugin_dest = "/opt/sonar/extensions/plugins"
def plugin_jar

node('linux-builder') {
  stage('checkout') {
    git "https://github.com/Westat-Transportation/sonar-r-plugin"
  }

  stage('build') {
    pom = readMavenPom file: "${env.WORKSPACE}/pom.xml"
    plugin_jar = "sonar-r-plugin-"+pom.version+".jar"
    echo pom.version
    echo "New plugin jar is ${plugin_jar}"
    sh "cd ${env.WORKSPACE}; ./mvnw clean package"
    stash includes: '.sonar/sonarqube-*/extensions/plugins/sonar-r-plugin*.jar', name: 'new-plugin-jar'
  }
}
node('master') {
  stage('deploy') {
    unstash 'new-plugin-jar'
    echo "Deploying $plugin_jar"
    sh """
        scp .sonar/sonarqube-*/extensions/plugins/sonar-r-plugin*.jar ${sonar_host}:
        ssh ${sonar_host} /bin/bash << EOF
           sudo -u sonarqube /bin/cp $plugin_dest/sonar-r-plugin*.jar /tmp
           sudo -u sonarqube /bin/rm $plugin_dest/sonar-r-plugin*.jar
           sudo -u sonarqube /bin/cp $plugin_jar $plugin_dest
           sudo /bin/systemctl restart sonar
           sudo /bin/systemctl status sonar
EOF
    """

  }
}
