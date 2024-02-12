pipeline{
    agent any
    environment{
        registry = "vdockh090/vproappcicd"
        registryCredential = 'dockerhub'
    }
    

    stages{
        stage('BUILD'){
            steps{
                sh 'mvn clean install -DskipTests'
            }
            post{
                success{
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps{
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success{
                    echo "Generated Analysis Result"
                }                
            }
        }

        stage('Build App Image'){
            steps{
                script{
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                    }
                }
            }
        

        stage('Upload Image'){
            steps{
                script{
                    docker.withRegistry('', registryCredential){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerimage.push('latest')
                    }
                }
            }
        }

        stage('Remove Unused Docker local image'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deploy'){
            agent{
                label 'KOPS'
            }

            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER}"
            }
        }

    }
}

