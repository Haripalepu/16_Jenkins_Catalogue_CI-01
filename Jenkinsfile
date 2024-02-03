//Angent should have Java,Terrafrom,Aws cli,node js
//The below stages are for Continues Integration process
//Stage-01 Get the version of the aplication.While doing the developments the application version will change (package.json file line no 3)
//Stage-02 Install Dependencies 
//Stage-03 Build i.e Zip all files(16_Jenkins_Catalogues) and create artifact
//Stage-04 Push Artifact to Nexus Repo
//Stage-05 Deploy. To pass the version & environmnet to the downstreat server.

pipeline {
    agent {
        node {
            label 'Agent' //Name of the Agent label 
        }
    }
    environment { 
        packageVersion = ''
        nexusURL = '44.210.142.46:8081' //Mention your Nexus Url
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()  //It won't allow us to run two builds at a time.
    }
    // build
    stages {
        stage('Get the version') { 
            steps {
                script { //Groovy scripting. def command is used declare a variable in pipeline.
                    def packageJson = readJSON file: 'package.json' //Now we will get all data from the package.json file and stores in packageJson by using Pipeline Utility Steps plugin.
                    packageVersion = packageJson.version //Now from that file we can take the version id. 
                    echo "application version: $packageVersion" //Calling from line no 12
                }
            }
        }
        stage('Install dependencies') {
            steps {     //Shell commands in pipeline. To run the below command Node Js should be installed in agent.
                sh """   
                    npm install  
                """
            }
        }
        stage('Build the zip file') {
            steps {   // zip is a linux command to zip the files -x to exclude the files while zipping, -q to hide the running log, it is unnecessary and waste of memory.
                sh """
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x "*.zip" 
                    ls -ltr
                """
            }
        }
        stage('Publish Artifact to Nexus Repo') { //Nexus artifact uploader plugin should be installed.
            steps {
                 nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusURL}",
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'catalogue',
                    credentialsId: 'Nexus_credentials', //Configure the Nexus credentials in jenkins and name it here
                    artifacts: [
                        [artifactId: 'catalogue',
                        classifier: '',
                        file: 'catalogue.zip',
                        type: 'zip']
                    ]
                )
            }
        }
        stage('Deploy') {
            steps {
                script {
                        def params = [
                            string(name: 'version', value: "$packageVersion"),
                            string(name: 'environment', value: "dev")
                        ]
                        build job: "catalogue-deploy", wait: true, parameters: params //Build job is to pass version & environment to catalogue-downstream server.
                    }               //catalogue-deploy is a pipeline name
            }
        }
     }
    // post build
    post { 
        always { 
            echo 'Deleting the workspace folder...!'
            deleteDir() //Once we create the zip file and store it nexus we have to delete the workspace folder.If not we face issue while running second time.
        }
        failure { 
            echo 'This runs when pipeline is failed, used generally to send some alerts'
        }
        success{
            echo 'I will say Hello when pipeline is success'
        }
    }
}