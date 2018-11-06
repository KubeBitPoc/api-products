#!/usr/bin/groovy

podTemplate(label: 'Jenkins', containers: [
   containerTemplate(name: 'jnlp', image: 'lachlanevenson/jnlp-slave:3.10-1-alpine', args: '${computer.jnlpmac} ${computer.name}', workingDir: '/home/jenkins', resourceRequestCpu: '200m', resourceLimitCpu: '300m', resourceRequestMemory: '256Mi', resourceLimitMemory: '512Mi'),
	containerTemplate(name: 'maven', image: 'jenkinsxio/builder-maven:0.1.22', command: 'cat', ttyEnabled: true),
    //containerTemplate(name: 'gradle', image: 'gradle-4.10.2-jdk7', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker:1.12.0', command: 'cat', ttyEnabled: true),    
	containerTemplate(name: 'kubectl', image: 'pahud/eks-kubectl-docker', command: 'cat', ttyEnabled: true),
	containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.8.2', command: 'cat', ttyEnabled: true)
],volumes:[
	hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/tmp/jenkins/.gradle'),
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),  
]) {
  node('Jenkins') {
    //def myRepo = checkout scm
    // def gitCommit = myRepo.GIT_COMMIT
    // def gitBranch = myRepo.GIT_BRANCH
    // def shortGitCommit = "${gitCommit[0..10]}"
    // def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
    // def GRADLE_USER_HOME=~/.gradle
    // def gradle_user_home='~/.gradle'
      
    // Build sucess
    /*stage('Test') {
      try {
        container('gradle') {
          sh """
            //def gradle_home=pwd
            //def gradle_user_home=${gradle_home}
            echo "GIT_BRANCH=${gitBranch}" >> /etc/environment
            echo "GIT_COMMIT=${gitCommit}" >> /etc/environment
            gradle test
            """
        }
      }
      catch (exc) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    */
    checkout scm
    stage('Build') {
      container('maven') {
        sh "echo $pwd"
        //sh "echo  'The current contents are $ls'"
        sh "mvn clean install"
      }
    }

    //pull the code at this location to build and deploy
    

    stage('Create Docker images') {
      container('docker') {
       
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockercredentials',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
           
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            echo "We are currently in directory $pwd"
			      docker build -t anilbb/api-products .
            docker push anilbb/api-products
            """
        }
      }
    }
    stage('Run kubectl') {
      container('kubectl') {
        sh "kubectl get pods"
      }
    }
    stage('Run helm') {
      container('helm') {
        sh "helm list"
        
          try{
            sh "helm del --purge api-products-helm"
          }
          catch(error) {
              echo "No previous helm deployments found"
          }
          
          
          echo "Removing the current helm chart package"
       
          try{
                sh "rm -f helm/api-products-helm-0.1.0.tgz"
          }catch(error) {
              echo "No previous helm package found"
          }
          
          
          echo "Creating the new helm package"
          try {  
            sh "helm package helm/api-products-helm" 
          } catch(error) {
              echo "created the package"
          }
          sh "cp api-products-helm-0.1.0.tgz helm/"
          echo "Installing the new helm package"
          
        sh "helm install --name api-products-helm helm/api-products-helm-0.1.0.tgz"
        
        echo "Application anilbb/api-products successfully deployed. Use helm status anilbb/api-products to check"
      }
    }
  }
}

