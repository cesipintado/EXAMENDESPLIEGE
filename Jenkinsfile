pipeline {
    agent any

    tools {
        // Instala la versión de Maven configurada como "MAVEN_HOME" y agrégala al path
        maven "MAVEN_HOME"
    }

    environment {
        // Variables de entorno útiles
        MAVEN_OPTS = '-Xmx1024m'
    }

    stages {
        stage('Clone') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Clona el repositorio desde GitHub
                    git branch: 'main', url: 'https://github.com/cesipintado/EXAMENDESPLIEGE.git'
                }
            }
        }
        
        stage('Build') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Compila el proyecto y genera el JAR
                    sh "mvn clean compile"
                }
            }
        }
        
        stage('Test') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    // Ejecuta las pruebas unitarias
                    sh "mvn test"
                }
            }
            post {
                always {
                    // Publica los resultados de las pruebas
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Genera el archivo JAR ejecutable
                    sh "mvn package -DskipTests"
                }
            }
            post {
                success {
                    // Archiva el JAR generado
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    withSonarQubeEnv('sonarqube') {
                        // Ejecuta el análisis de código con SonarQube
                        sh """
                            mvn sonar:sonar \
                                -Dsonar.projectKey=CursoDespliegue \
                                -Dsonar.projectName='Curso Despliegue' \
                                -Dsonar.projectVersion=1.0 \
                                -Dsonar.sources=src/main/java \
                                -Dsonar.tests=src/test/java \
                                -Dsonar.java.binaries=target/classes \
                                -Dsonar.junit.reportsPath=target/surefire-reports \
                                -Dsonar.jacoco.reportsPath=target/site/jacoco/jacoco.xml
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Espera y verifica que el proyecto pase el Quality Gate de SonarQube
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Deploy Preparation') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        // Verifica que el JAR se haya generado correctamente
                        sh "ls -la target/*.jar"
                        echo "JAR file ready for deployment"
                        
                        // Opcional: Mostrar información del JAR
                        sh "java -jar target/*.jar --version || echo 'Version info not available'"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Limpia el workspace después de la ejecución
            cleanWs()
        }
        success {
            echo "Pipeline executed successfully! 🎉"
            echo "Project approved by SonarQube and ready for deployment."
        }
        failure {
            echo "Pipeline failed! ❌"
            echo "Check the logs above for details."
        }
        unstable {
            echo "Pipeline completed but with warnings ⚠️"
        }
    }
}