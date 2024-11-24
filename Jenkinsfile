pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven'
    }
    environment{
        SONAR_HOME = tool 'sonar-scanner'
    }
    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Tag for the Docker image')
        
    }


    stages {
        stage('Clone the Project') {
            steps {
                git 'https://github.com/udayyysharmaa/Multi-Tier-BankApp-CI.git'
            }
        }
        stage('Compile the project') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test the Project') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('Scan the File') {
            steps {
                sh 'trivy fs --format json -o index.json .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=bankapp -Dsonar.projectKey=bankapp -Dsonar.java.binaries=. "
                    
                }
            }
        }

        stage('Quality Gate ') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Build the Project') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('Push the Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'uday', jdk: 'jdk17', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('Create a Docker Image') {
            steps {
                script{
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh " docker build -t onlinelearningofficial/bankapp:${params.DOCKER_TAG} ."
                    }
                }
            }
        }
        stage('Scan the docker image') {
            steps {
                sh "trivy image --format json -o image.json onlinelearningofficial/bankapp:${params.DOCKER_TAG}"
            }
        }
        stage('Push to docker hub') {
            steps {
                script{
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push onlinelearningofficial/bankapp:${params.DOCKER_TAG} "
                    }
                }
            }
        }
        stage('Update YAML Manifest in Other Repo') {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                    script {
                        // Check if the directory already exists
                        if (fileExists('Multi-Tier-BankApp-CD')) {
                            // If it exists, clean it up
                            sh 'rm -rf Multi-Tier-BankApp-CD'
                            
                        }
                        
                    }
                    sh '''
                    # Clone the repo containing the Kubernetes deployment manifest
                    git clone https://github.com/udayyysharmaa/Multi-Tier-BankApp-CD.git
                    cd Multi-Tier-BankApp-CD
                    # Check if the bankapp-ds.yml file exists
                    ls -l bankapp
                    # Update the Docker image tag in the YAML file
                    sed -i 's|image: onlinelearningofficial/bankapp:.*|image: onlinelearningofficial/bankapp:'${DOCKER_TAG}'|' $(pwd)/bankapp/bankapp-ds.yml
                    '''
                    // Confirm the change in the updated YAML file
                    sh '''
                    echo "Updated YAML file contents:"
                    cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                    '''
                    // Configure Git for the commit
                    sh '''
                    cd Multi-Tier-BankApp-CD
                    git config user.email "udaysharmaniit12345@gmail.com"
                    git config user.name "udayyysharmaa"
                    '''
                    // Commit and push the changes back to the repository
                    sh '''
                    cd Multi-Tier-BankApp-CD
                    git add bankapp/bankapp-ds.yml
                    git commit -m "Update image tag to ${DOCKER_TAG}"
                    git push origin master
                    '''
                    
                }
                
            }
            
        }
        
    }
}




