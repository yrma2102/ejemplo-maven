import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        NOMBRE_ALUMNO = "Yrma Ccencho"
        PERSONAL_CHANNEL= credentials('personal-channel')
        NEXUS_PASSWORD = credentials('usuario-nexus')
    }
    stages {
        stage("Paso 1: Build && Test"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh "echo 'Build && Test!'"
                    sh "./mvnw clean package -e"    
                }
            }
        }

        stage("Paso 2: Sonar - Análisis Estático"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh "echo 'Análisis Estático!'"
                        withSonarQubeEnv('sonarqube') {
                            sh "echo 'Calling sonar by ID!'"
                            // Run Maven on a Unix agent to execute Sonar.
                            sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo-maven-full-stages -Dsonar.projectName=cejemplo-maven-full-stages -Dsonar.java.binaries=build'
                        }
                        
                }
            }
        }
        
        stage("Paso 3: Curl Springboot maven sleep 20"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh "nohup bash ./mvnw spring-boot:run  & >/dev/null"
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
        stage("Paso 4: test newman"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh 'newman run ejemplo-maven2.postman_collection.json'
                }
            }
        }
        stage("Paso 5: Detener Spring Boot"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh '''
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')  
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }
           stage("Paso 6: Subir Artefacto a Nexus"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-usach-ceres',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }
        stage("Paso 7: Descargar Nexus"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh ' curl -X GET -u $NEXUS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            }
        }
         stage("Paso 8: Levantar Artefacto Jar en server Jenkins"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
          stage("Paso 9: Testear Artefacto - Dormir(Esperar 20sg) "){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
        stage("Paso 10: test newman to jar from nexus"){
            steps {
                script{
                    NOMBRE_STAGE = env.STAGE_NAME
                    sh 'newman run ejemplo-maven2.postman_collection.json -n 2 --delay-request 10'
                }
            }
        }
        stage("Paso 11:Detener Atefacto jar en Jenkins server"){
            steps {
                script{
                     NOMBRE_STAGE = env.STAGE_NAME
                sh '''
                    echo 'Process Java .jar: ' $(pidof java | awk '{print $1}')  
                    sleep 20
                    kill -9 $(pidof java | awk '{print $1}')
                '''
                }
               
            }
        }
    }
    post {
        
            success {
                slackSend color : "good", message: "Build Success: [${NOMBRE_ALUMNO}][JOB : ${env.JOB_NAME}][buildTool : ${env.BUILD_TAG}] Ejecución exitosa", channel: "${PERSONAL_CHANNEL}"
            }
            failure {
                slackSend color : "danger", message: "Build Failure:[${NOMBRE_ALUMNO}][JOB : ${env.JOB_NAME}][buildTool : ${env.BUILD_TAG}] Ejecución fallida en stage [${NOMBRE_STAGE} ]", channel: "${PERSONAL_CHANNEL}"
            }
            
    }
}
