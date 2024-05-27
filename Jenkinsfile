node {
    
    def application = "devopsexample"
    
    // It's mandatory to change the Docker Hub Account ID after this Repo is forked by another person
    def dockerhubaccountid = "dockersv2"
    
    // Reference to maven
    // ** NOTE: This 'maven-3.5.2' Maven tool must be configured in the Jenkins Global Configuration.   
    def mvnHome = tool 'maven-3.5.2'

    // Holds reference to docker image
    def dockerImage
 
    // Constructing docker image tag with job name and build number
    def dockerImageTag = "${dockerhubaccountid}/${application}:${env.JOB_NAME.replace('/', '_')}_${env.BUILD_NUMBER}"
    
    stage('Clone Repo') { 
        // Get some code from a GitHub repository
        git url:'https://github.com/sohelkhan121101/jenkins-demo.git', branch:'main' // Update your forked repo
        // Get the Maven tool.
        // ** NOTE: This 'maven-3.5.2' Maven tool must be configured
        // ** in the global configuration.           
        mvnHome = tool 'maven-3.5.2'
    }    
  
    stage('Build Project') {
        // Build project via maven
        sh "'${mvnHome}/bin/mvn' clean install"
    }
        
    stage('Build Docker Image with new code') {
        // Build docker image
        dockerImage = docker.build("${dockerhubaccountid}/${application}:${env.BUILD_NUMBER}")
    }
    
    // Push image to remote repository, in your Jenkins you have to create the global credentials similar to the 'dockerHub' (credential ID)
    stage('Push Image to Remote Repo') {
        echo "Docker Image Tag Name ---> ${dockerImageTag}"
        docker.withRegistry('', 'dockerHub') {
            dockerImage.push("${env.BUILD_NUMBER}")
            dockerImage.push("latest")
        }
    }
   
    stage('Remove running container with old code') {
        // Remove the container which is already running, when running for the first time named container will not be available so we are using 'True'
        // Added -a option to remove stopped container also
        sh "docker rm -f \$(docker ps -a -f name=devopsexample -q) || true"   
    }
    
    stage('Deploy Docker Image with new changes') {
        // Start container with the remote image
        sh "docker run --name devopsexample -d -p 2222:2222 ${dockerhubaccountid}/${application}:${env.BUILD_NUMBER}"  
    }
    
    stage('Remove old images') {
        // Remove old docker images
        sh("docker rmi ${dockerhubaccountid}/${application}:latest -f")
   }
}
