def ReleaseDir = "c:\\aplicativos\\AspnetApp"

pipeline {
   agent any   
   options {
	    timeout(time: 1, unit: 'DAYS')
        buildDiscarder(logRotator(numToKeepStr: '4', artifactNumToKeepStr: '4')) 
        skipDefaultCheckout()
      }
   stages {
      stage('Checkout SCM') {
         steps {
            checkout scmGit(
               branches: [[name: 'master']], 
               browser: github('https://github.com/rayssa-amorim/aspnet-app.git'), 
               extensions: 
                    [cloneOption(honorRefspec: true, noTags: true, reference: '', shallow: false), lfs(), localBranch('master')], 
               userRemoteConfigs: [[credentialsId: 'Github', url: 'https://github.com/rayssa-amorim/aspnet-app.git']])
         }
      }
      stage('Compilacao da aplicacao Aspnet') {
         steps {
            bat """
               C:\\Ferramentas\\Nuget\\nuget.exe restore src\\aspnetapp.sln
            """
           
            }
         }
         stage('Pre-deploy da aplicacao') {
 			   steps {
              catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                  script{
                  try {
                    bat "\"${tool 'Msbuild'}\" src\\aspnetapp.sln /p:DeployOnBuild=true /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:SkipInvalidConfigurations=true /t:build -restore /p:Configuration=Release /p:Platform=\"Any CPU\" /p:DeleteExistingFiles=True /p:publishUrl=${ReleaseDir}"
                 }
                 catch (Exception e) {
                   buildAborted = true
						 currentBuild.result = 'FAILURE'
						 error 'Erro na compilacao: ' + e.toString()
                  
                    }
                  
                  }
                           
                }
 					   
             }
 		   }	
         stage('Execucao Ansible-playbook para deploy aplicacao Aspnet') {
            agent { label 'linux-ansible'} 
            steps{
                ansiblePlaybook colorized: true,
                credentialsId: 'jenkinsAnsible', disableHostKeyChecking: true, installation: 'ansible',
                inventory: '/etc/ansible/inventory/hosts', playbook: '/etc/ansible/playbooks/WindowsTest.yml'
            }
         }
     } 
  }    
   
