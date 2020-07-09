node('maven') {
    stage('SCM') {
    // clone source code
    git 'https://github.com/ABBANAPURI0445/game-of-life.git'
}
 stage('Build') {
    // build java source code using maven build tool mvn compile + mvn test + mvn package
    sh label: '', script: 'mvn package'
 }
 stage('Junit'){
     // Publish Junit Results
     junit 'gameoflife-web/target/surefire-reports/*.xml'
 }
 stage('Archive the artifact'){
     // store artifact in local machine or shared folder
  archive 'gameoflife-web/target/gameoflife.war'
 }
 stage('Static Code analysis'){
       // performing sonarqube analysis with "withSonarQubeENV(<Name of Server configured in Jenkins>)"
    withSonarQubeEnv('sonar') {
      // requires SonarQube Scanner for Maven 3.2+
      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
    }
 }
 stage('Artifactory'){
     withCredentials([usernamePassword(credentialsId: 'jfrog', passwordVariable: 'password', usernameVariable: 'user')]) {
    sh "curl -u $user:$password -T /home/maven/workspace/CICD/gameoflife-web/target/*.war http://34.211.148.219:8081/artifactory/gameoflife/gameoflife${BUILD_NUMBER}.war"
}
     
 }
}
// CD
node('docker') {
   stage('clone dockerfile') {
    git 'https://github.com/ABBANAPURI0445/docker-ursi.git'
}
  stage('build docker image'){
    withCredentials([usernamePassword(credentialsId: 'jfrog', passwordVariable: 'password', usernameVariable: 'user')]) {
    sh "docker image build --build-arg user=$user --build-arg password=$password --build-arg url=http://34.211.148.219:8081/artifactory/gameoflife/gameoflife${BUILD_NUMBER}.war -t gol:${BUILD_NUMBER}.0 ."
}  
  }
  stage('docker login'){
      withCredentials([usernamePassword(credentialsId: 'Dockerid', passwordVariable: 'dockerhubpassword', usernameVariable: 'dockerhubuser')]) {
     sh "docker login -u $dockerhubuser -p $dockerhubpassword"
}
  }
 stage('docker image push'){
    sh label: '', script: '''docker tag gol:${BUILD_NUMBER}.0 abbanapuri0445/eks:${BUILD_NUMBER}.0
docker push abbanapuri0445/eks:${BUILD_NUMBER}.0'''
  }
  }
  // Kuberntes write a manifest file and push to github
node('kubectl') {
    stage('clone-manifestfile') {
   git 'https://github.com/ABBANAPURI0445/k8s-manifestfile.git'
}
stage('deploy'){
    sh label: '', script: '''kubectl apply -f deployment.yaml
kubectl apply -f service-lb.yaml'''
}
// below stage is for deploy newer version  --> general this stage is separate 
stage('rollout'){
    sh "kubectl --record deployment.apps/deploy-appserver set image  appserver=abbanapuri0445/ursi:${BUILD_NUMBER}.0"
}
}
