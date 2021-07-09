pipeline {
    agent any
    tools {
        maven 'Maven 3.6.1'
    }
    parameters {
        string(defaultValue: "", description: 'String used to tag and release emissary if pipeline successfully builds.', name: 'TAG_EMISSARY')
	string(defaultValue: "default", description: 'String used as git username for tag and release purposes and credentials to do so.', name:'GIT_NAME')
    }
    stages {
        stage('Build Emissary') {
            steps {
		sh 'sudo yum install -y java-1.8.0-openjdk-devel java-1.8.0-openjdk tar expect which docker docker-compose'
		sh 'cat src/test/resources/jenkins_config/bashrc_addition >> ~/.bashrc'
		sh 'mkdir -p ~/.m2'
		sh 'sudo chown -R $(whoami):$(whoami) $(pwd)'
		sh 'rm -rf ~/.m2/repository'
		sh 'mvn clean install'
		sh 'mvn clean compile'
		sh ('if [ "$TAG_EMISSARY" != "" ]; then mvn versions:set -DnewVersion=${TAG_EMISSARY}; fi;')
		sh ('if [ "$TAG_EMISSARY" != "" ]; then mvn versions:commit; fi;')
            }
        }
	stage('Build Docker Images'){
	    steps {
	        sh 'sudo systemctl start docker'
		sh 'mvn clean package -Pdist'
	   	sh ('if [ "$TAG_EMISSARY" == "" ]; then docker build -t emissary:latest --build-arg PROJ_VERS=$(./emissary version | grep Version: | awk {\'print $3 " " \'}) --build-arg IMG_NAME=latest .; else docker build -t emissary:${TAG_EMISSARY} --build-arg PROJ_VERS=${TAG_EMISSARY} --build-arg IMG_NAME=${TAG_EMISSARY} .; fi;')
		sh ('if [ "$TAG_EMISSARY" == "" ]; then docker build -f Dockerfile-test-feeder -t emissary-test-feeder:latest --build-arg PROJ_VERS=$(./emissary version | grep Version: | awk {\'print $3 " " \'}) --build-arg IMG_NAME=latest .; else docker build -f Dockerfile-test-feeder -t emissary-test-feeder:${TAG_EMISSARY} --build-arg PROJ_VERS=${TAG_EMISSARY} --build-arg IMG_NAME=${TAG_EMISSARY} .; fi;')
	    }
	}
        stage('Test') {
            steps {
	    	sh 'mvn test'
		sh 'rm -f test_results'
		sh 'sudo ./Docker_compose_test_script.sh'
            }
        }
	stage('Tag + Release'){
	    steps {
	        sh ('if [ "$GIT_NAME" != "default" ]; then git config user.name "${GIT_NAME}"; fi;')
	    	withCredentials([usernamePassword(credentialsId: env.GIT_NAME , usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
		    sh ('if [ "$GIT_NAME" != "default" -a "$TAG_EMISSARY" != "" ]; then git add .; fi;')
		    sh ('if [ "$GIT_NAME" != "default" -a "$TAG_EMISSARY" != "" ]; then git commit -m "Commit made from Jenkins for tag ${TAG_EMISSARY}"; fi;')
		    sh ('if [ "$GIT_NAME" != "default" -a "$TAG_EMISSARY" != "" ]; then git tag -a ${TAG_EMISSARY} -m "Jenkins tag of ${TAG_EMISSARY}"; fi;')
		    sh ('if [ "$GIT_NAME" != "default" -a "$TAG_EMISSARY" != "" ]; then git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/NationalSecurityAgency/emissary.git --tags; fi;')
		}
	    }
	}
    }
}