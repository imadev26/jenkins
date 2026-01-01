pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=C:\\ProgramData\\Jenkins\\.jenkins\\.m2\\repository"
    }

    stages {

        stage('Cloner le dépôt') {
            steps {
                echo 'Clonage du dépôt GitHub...'
                git branch: 'main', url: 'https://github.com/lachgar/jenkins2.git'
            }
        }

        stage('Build and SonarQube Analysis') {
            parallel {

                stage('Car Service') {
                    stages {

                        stage('Build Car Service') {
                            steps {
                                dir('car') {
                                    echo 'Compilation et génération du service Car...'
                                    script {
                                        bat 'mvn clean install -DskipTests'
                                    }
                                }
                            }
                        }

                        stage('SonarQube Analysis Car Service') {
                            steps {
                                dir('car') {
                                    script {
                                        def mvn = tool 'maven';
                                        withSonarQubeEnv('SonarQube-Car') {
                                            bat "${mvn}\\bin\\mvn clean verify ^ " +
                                                "sonar:sonar ^ " +
                                                "-Dsonar.projectKey=car ^ " +
                                                "-Dsonar.projectName='car' ^ " +
                                                "-DskipTests"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

                stage('Client Service') {
                    stages {

                        stage('Build Client Service') {
                            steps {
                                dir('client') {
                                    echo 'Compilation et génération du service Client...'
                                    script {
                                        bat 'mvn clean install -DskipTests'
                                    }
                                }
                            }
                        }

                        stage('SonarQube Analysis Client Service') {
                            steps {
                                dir('client') {
                                    script {
                                        def mvn = tool 'maven';
                                        withSonarQubeEnv('SonarQube-Client') {
                                            bat "${mvn}\\bin\\mvn clean verify ^ " +
                                                "sonar:sonar ^ " +
                                                "-Dsonar.projectKey=client ^ " +
                                                "-Dsonar.projectName='client' ^ " +
                                                "-DskipTests"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

                stage('Gateway Service') {
                    steps {
                        dir('gateway') {
                            echo 'Compilation et génération du service Gateway...'
                            script {
                                bat 'mvn clean install -DskipTests'
                            }
                        }
                    }
                }

                stage('Eureka Server') {
                    steps {
                        dir('server_eureka') {
                            echo 'Compilation et génération du serveur Eureka...'
                            script {
                                bat 'mvn clean install -DskipTests'
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Compose') {
            steps {
                dir('deploy') {
                    echo 'Création et déploiement des conteneurs Docker...'
                    script {
                        echo 'Arrêt et suppression des conteneurs existants...'
                        bat '''
                            docker-compose down --remove-orphans 2>nul || echo "No compose containers to remove"
                            docker rm -f consul-container 2>nul || echo "consul-container not found"
                            docker rm -f mysql-container1 2>nul || echo "mysql-container1 not found"
                            docker rm -f phpmyadmin-container 2>nul || echo "phpmyadmin-container not found"
                            docker rm -f eureka-server 2>nul || echo "eureka-server not found"
                            docker rm -f gateway-service 2>nul || echo "gateway-service not found"
                            docker rm -f client-service 2>nul || echo "client-service not found"
                            docker rm -f voiture-service 2>nul || echo "voiture-service not found"
                        '''
                        echo 'Démarrage des conteneurs...'
                        bat 'docker-compose up -d --build'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline exécuté avec succès !'
        }
        failure {
            echo 'Le pipeline a échoué. Vérifiez les logs.'
        }
    }
}
