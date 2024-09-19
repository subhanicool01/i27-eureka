pipeline {
    agent {
        label 'k8s-slave'
    }
    environment{
        SERVICE_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "index.docker.io/subhanicool01"
        DOCKER_CREDS = credentials('subhanicool01_docker_creds')
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    stages {
        stage("Build") {
            steps {
                echo "Starting the build"
                echo "Building the code for ${env.SERVICE_NAME}"
                echo "Now Starting the mvn packages"
                sh "mvn clean package -DskipTests=True"
                echo "Build is done"             
            }
        }
        stage('Unit tests') {
            steps {
                echo "performing the unit test cases"
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Docker format') {
            steps {
                echo "actual format: ${env.SERVICE_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
                //Need to have below formating
                //service-name-buildnumber-branchname.packing
                //pricecur-service-5-main.jar
                echo "custom format: ${env.SERVICE_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_PACKAGING}"
            }
        }
        stage('Docker build') {
            steps {
                sh """
                  ls -la
                  cp -f ${workspace}/target/i27-${env.SERVICE_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                  ls -la ./.cicd
                  echo "********************** Build docker image ************************"
                  docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.SERVICE_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.SERVICE_NAME}:${GIT_COMMIT} ./.cicd
                  docker images
                  echo "*********************** Docker push *****************************"
                  docker login ${env.DOCKER_HUB} -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                  docker push  ${env.DOCKER_HUB}/${env.SERVICE_NAME}:${GIT_COMMIT}
                """
            }
        }
    }
}

//  /home/subhanicool01/jenkins/workspace/pricecut-service_main/target/price-cut-service-0.0.1-SNAPSHOT.jar
//   git remote set-url origin https://{subhanicool01}:{Subhani@5736}@github.com/{subhanicool01}/project.git
//   after install the docker we need to fix one issue for permission denied. To add user for docker group so we write a module on ansible playbook.

//docker build --force-rm --no-cache --pull --rm=true 
// --force-rm - forcefully remove
// --no-cache - no cacheing 
// --pullm --rm=true - to pull the remote directory
