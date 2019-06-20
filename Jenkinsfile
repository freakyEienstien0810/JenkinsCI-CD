pipeline{
    agent{
        // Note - This could change depending upon the agent who has to run the job
        label "master"
    }

    tools{
        // Note - This should matchc with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        jdk "JAVA_HOME"
        maven "Maven-3.6.1"
    }

    environment{
        //This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        //This can be http or https
        NEXUS_PROTOCOL = "http"
        //Where your nexus is running
        NEXUS_URL = "192.168.30.154:8081"
        //Jenkins credential id to authneticate to Nexus OSS/PRO
        NEXUS_CREDENTIAL_ID = "nexus-anirudha-cred"
        //Nexus Repository targeted for artifcats upload
        NEXUS_REPOSITORY = "dev-releases"
        //Nexus task ID, this task rebuilds nexus repo metadata
        NEXUS_TASK_ID = "5f1249c1-e5c3-4820-a023-dbd3e19f5194"
    }

    stages{
           stage("Checking out code from GIT repo and running mvn build"){
                steps{
                    bat "mvn clean package"
                }
                post{
                    success{
                        echo "======== mvn build executed successfully ========"
                    }
                    failure{
                        echo "======== mvn build execution failed please check the logs for further information ========"
                    }
                }
            }
            stage("Publish Artifacts to Nexus"){
                steps{
                    script{
                        // Read POM xml file using 'readMavenPom' step, this step us 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                        def pom = readMavenPom file: 'pom.xml';
                        // Find built artifact underf target folder
                        def filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                        // Printing info from the artifact found
                        echo " ${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                        // Extracting the path from the file found
                        def artifactPath = filesByGlob[0].path;
                        // Assigning to a boolean response verfiying if the artifact name exists
                        def artifactExists = fileExists artifactPath;
                        // Checking if artifact exists before upload proccess to nexus OSS/PRO repo
                        if(artifactExists){
                            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version: ${pom.version}"
                            
                            // Setting Environment Varibles for future usages
                            env.PACKAGE_VERSION =  "${pom.packaging}"
                            env.PACKAGE_groupID = "${pom.groupId}"
                            env.PACKAGE_artifcatID = "${pom.artifactId}"

                            // Upload using nexusArtifaactUploader step, this step is in: https://plugins.jenkins.io/nexus-artifact-uploader
                            nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: pom.version,
                                repository: NEXUS_REPOSITORY,
                                credentialsId: NEXUS_CREDENTIAL_ID,
                                artifacts: [
                                    // Artifact generated such as .jar, .ear and .war files.
                                    [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging],
                                    // Upload the pom.xml file for additional information for Transitive dependencies
                                    [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"]
                                ]
                            );                        
                        }else {
                            error "*** File: ${artifactPath}, could not be found"
                        }

                    }
                }
                post{
                    success{
                        echo "==== Upload to nexus OSS/PRO Repo successfull ===="
                    }
                    failure{
                        echo "==== Upload failed please look into the logs for more information ===="
                    }
            
                }
            }
            stage("Update Nexus MetaData file"){
                steps {
                    withCredentials([usernameColonPassword(credentialsId: NEXUS_CREDENTIAL_ID, variable: 'USERPASS')]) {
                        bat 'curl -u %USERPASS% -X POST \"%NEXUS_PROTOCOL%://%NEXUS_URL%/service/rest/v1/tasks/%NEXUS_TASK_ID%/run\"'
                    }
		    sleep(10);
    	        }    
                post{
                    success{
                        echo "==== Update Nexus MetaData file executed succesfully ===="
                    }
                    failure{
                        echo "==== Update Nexus MetaData file execution failed ===="
                    }
            
                }
            }
            stage("Deployment through IBM UrbanCode Deploy"){
                steps{
                    step([$class: 'UCDeployPublisher',
                        siteName: 'UCD-v7.0.1',
                        component: [
                            $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
                            componentName: 'NexusTest',
                            delivery: [
                                $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Pull',
                            ]
                        ]
	                ])

                    sleep(10);
                    step([$class: 'UCDeployPublisher',
                        siteName: 'UCD-v7.0.1',
                        deploy: [
                            $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeployHelper$DeployBlock',
                            deployApp: 'DummyDeploymentLocal',
                            deployEnv: 'Dev',
                            deployProc: 'DummyDeploymentLocal',
                            deployVersions: 'NexusTest:'+"${env.PACKAGE_VERSION}",
                            deployOnlyChanged: false
                        ]
                    ])
                }
                post{
                    success{
                        echo "====++++Deployment through IBM UrbanCode Deploy executed succesfully++++===="
                    }
                    failure{
                        echo "====++++Deployment through IBM UrbanCode Deploy execution failed++++===="
                    }
            
                }
            }
        }
        post{
            success{
                echo "======== Pipeline executed successfully ========"
            }
            failure{
                echo "======== Pipeline execution failed ========"
            }
        }
}
