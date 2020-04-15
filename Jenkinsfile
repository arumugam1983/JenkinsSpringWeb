node {
  def mvnHome
  def pom
  def artifactVersion
  def tagVersion
  def retrieveArtifact

  stage('Prepare') {
    echo 'Start Prepare'
    mvnHome = tool 'MavenDefault'
    echo 'End Prepare'
  }

  stage('Checkout') {
     checkout scm
  }

  stage('Build') {
     if (isUnix()) {
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
     } else {
        bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
     }
  }

  stage('Unit Test') {
     junit '**/target/surefire-reports/TEST-*.xml'
     archive 'target/*.jar'
  }

  stage('Integration Test') {
    if (isUnix()) {
       sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean verify"
    } else {
       bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean verify/)
    }
  }

  stage('Sonar') {
     if (isUnix()) {
        sh "'${mvnHome}/bin/mvn' sonar:sonar"
     } else {
        bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
     }
  }

  if(env.BRANCH_NAME == 'master'){
    stage('Validate Build Post Prod Release') {
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" clean package/)
      }
    }

  }

f(env.BRANCH_NAME == 'develop'){
    stage('Snapshot Build And Upload Artifacts') {
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' clean deploy"
      } else {
	  
	  rtServer (
    id: 'Artifactory-1',
    url: 'http://localhost:8081/artifactory/libs-snapshot-local',
	
    // If you're using username and password:
    username: 'admin',
    password: 'password'
    // If you're using Credentials ID:
    credentialsId: 'ccrreeddeennttiiaall'
    // If Jenkins is configured to use an http proxy, you can bypass the proxy when using this Artifactory server:
    bypassProxy: true
    // Configure the connection timeout (in seconds).
    // The default value (if not configured) is 300 seconds:
    timeout = 300
)

    stage('Deploy') {
       sh 'curl -u jenkins:jenkins -T target/**.war "http://localhost:9090/manager/text/deploy?path=/devops&update=true"'
    }

    stage("Smoke Test"){
       sh "curl --retry-delay 10 --retry 5 http://localhost:9090/devops"
    }

  }

}
