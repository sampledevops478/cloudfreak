pipeline {
  agent any
  //parameters {
  //gitParameter branch: '', branchFilter: '.*', defaultValue: 'pprd', description: 'Branch to select', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'ASCENDING_SMART', tagFilter: '*', type: 'PT_BRANCH'
  //gitParameter branch: '', branchFilter: '.*', defaultValue: '1.0', description: 'Tag to build', name: 'GIT_TAG', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'ASCENDING_SMART', tagFilter: '*', type: 'PT_TAG'
  //}
  parameters {
  gitParameter branch: '', branchFilter: '.*', defaultValue: 'origin/master', description: 'Select branch from below drop down list', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', type: 'PT_BRANCH'
  string defaultValue: '1.0', description: 'Git tag release version to build', name: 'GIT_TAG', trim: false
  }	
    tools {
      maven 'maven3'
                 jdk 'JDK8'
    }
    stages {
        stage('clone'){
		    steps {
			  //checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: 'https://github.com/sampledevops478/cloudfreak.git', credentialsId: '976c8a11-ab25-4ef8-9344-39d3de2f67db' ]], branches: [[name: '+refs/heads/*:refs/remotes/$BRANCH +refs/tags/*:refs/tags/$GIT_TAG']]], poll: false
			    checkout([$class: 'GitSCM', branches: [[name: '${BRANCH}']], extensions: [], userRemoteConfigs: [[credentialsId: '976c8a11-ab25-4ef8-9344-39d3de2f67db', url: 'https://github.com/sampledevops478/cloudfreak.git']]])
			}
        }
        stage('Build maven') {
            steps { 
                    sh 'pwd'      
                    sh 'mvn  clean install package'
            }
        }
        stage('Copy Artifact') {
           steps { 
                   sh 'pwd'
                   sh 'rm -rf docker/*.jar'
		           sh 'cp -r target/*.jar docker'
           }
        }
         
        stage('Build docker image') {
           steps {
               script {         
                 // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
				 pom = readMavenPom file: "pom.xml";
                 def customImage = docker.build('444078758910.dkr.ecr.us-east-1.amazonaws.com/ull-docker-images' + ":v${pom.version}", "./docker")
                 docker.withRegistry('https://444078758910.dkr.ecr.us-east-1.amazonaws.com/', 'ecr:us-east-1:ecr-cred') {
                 customImage.push("v${pom.version}")
                 }                     
            }
        }
	  }
    }
    post
    {
        always
        {
            //make sure that the Docket image is removed
            sh "docker rmi  444078758910.dkr.ecr.us-east-1.amazonaws.com/ull-docker-images:v${pom.version} | true"
            sh "docker rmi  444078758910.dkr.ecr.us-east-1.amazonaws.com/ull-docker-images:latest | true"
        }    
    }
}
