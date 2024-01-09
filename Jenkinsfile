pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/pushkarshajapurkar/BoardgameListingWebApp.git', branch: 'master'
            }
        }
        
        stage('Clean') {
            steps {
                sh "mvn clean"
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Package') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame \
                        -Dsonar.projectKey=Boardgame123 \
                        -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        
        stage('Deploy Artifact To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: false) {
                    nexusArtifactUploader artifacts: [[artifactId: 'database_service_project', classifier: '', file: '/var/lib/jenkins/workspace/Corporate-Pipeline/target/database_service_project-0.0.2.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.javaproject', nexusUrl: '3.88.160.212:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: '0.0.2'
                }
                    
            }
        }
        
        stage('Image Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DOCKER', toolName: 'docker') {
                        sh "docker build . -t boardgamewebapp:latest"
                        sh "docker tag boardgamewebapp:latest pushkar4399/boardgamewebapp:latest"
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image pushkar4399/boardgamewebapp:latest"
            }
        }
        
        stage('Image Push To DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DOCKER', toolName: 'docker') {
                        sh "docker push pushkar4399/boardgamewebapp:latest"
                    }
                }
            }
        }
        
        // stage('Deploy To Container') {
        //     steps {
        //         script {
        //             withDockerRegistry(credentialsId: 'DOCKER', toolName: 'docker') {
        //                 sh "docker run -d -p 8085:8080 pushkar4399/boardgamewebapp:latest"
        //             }
        //         }
        //     }
        // }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(caCertificate: '''-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTI0MDEwOTA5NTIzM1oXDTM0MDEwNjA5NTIzM1owFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMP4
XoyK5DAG1iuih82MlF1LCOZS12n1RaJg8FjICmrBPoV1rx/ZnfDfVIhcsFzThf2o
bAxEuXfEkRwo4w2wP9QZnHV7H2ScXs+Nw8usgpI52+GsGswyqq84pklpdinRtQPB
P1D/ffQzfrbI0wN7EjE18WYQweHVbxg5F4ug5Z7IRPBUp7M9kY4wm8dGR4t5BPVR
qDfPp90hzMyVZkTXdv8jKleY+ygeyaNq9j1AmhUWKZy2YoFwVCbmPSfCvy/5A7iu
ktwBjBvvuh0hxgecdoEdZRvo44zru3EpS0bu+vTkxAZwQVsnx81JqVcGG4KpYXj/
Wtrj2ByQFP006YG20uECAwEAAaNCMEAwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wHQYDVR0OBBYEFDWTP5eV64Qb30fGnfo+Do3RJg/eMA0GCSqGSIb3
DQEBCwUAA4IBAQB895VmFO99laKzVj5wjZfmU7Pckbuc5DBhCs7+El25C+RMdcO2
cvzWxtKxLgElLLiivt6WoVUrOLG7Sdy3iQCAdL47OqMUMO+FvK0QyI0YOcC17rcp
fDk0VH5s51r0xcTMSMRWcTcK6DT9OpSfXDgljM6UV2HSlz3wrS1zYVdaG3YzUhKW
tdxdyX9mDHlX5onRucw5upgcXeuIZWzHY6JJPgVSUsCvbXSnBsLEcvqf+jgUcyvi
UzGAiXQv2wVbcNxaDp5BautVxyym4MrgkbEGFsHbrzPufKGhSiGex0UM1aa5G2LF
BdtL+uwfFOc/4e6KsEkioPwj/temBQCubcPE
-----END CERTIFICATE-----''', clusterName: '', contextName: '', credentialsId: 'k8s-secret', namespace: 'boards', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.27.245:6443') {
                  sh "kubectl apply -f deployment-service.yml"
                  sh "kubectl get pods -n boards"
                  sh "kubectl get svc -n boards"
            }
        }
                }
            }
    }
}
