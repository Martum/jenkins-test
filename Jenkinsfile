pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh '''#!/bin/bash +x
set -e

# Remove unnecessary files
echo -e "\\033[34mRemoving unnecessary files...\\033[0m"
rm -f log/*.log &> /dev/null || true &> /dev/null
rm -rf public/uploads/* &> /dev/null || true &> /dev/null

# Build Project
echo -e "\\033[34mBuilding Project...\\033[0m"
sudo docker-compose --project-name=${JOB_NAME} build

# Prepare test database
COMMAND="bundle exec rake db:drop db:create db:migrate"
echo -e "\\033[34mRunning: $COMMAND\\033[0m"
sudo docker-compose --project-name=${JOB_NAME} run  \\
	-e RAILS_ENV=test web $COMMAND

# Run tests
COMMAND="bundle exec rspec spec"
echo -e "\\033[34mRunning: $COMMAND\\033[0m"
sudo unbuffer docker-compose --project-name=${JOB_NAME} run web $COMMAND

# Run rubocop lint
COMMAND="bundle exec rubocop app spec -R --format simple"   
echo -e "\\033[34mRunning: $COMMAND\\033[0m"
sudo unbuffer docker-compose --project-name=${JOB_NAME} run -e RUBYOPT="-Ku" web $COMMAND
'''
      }
    }
    stage('Clean') {
      steps {
        sh '''#!/bin/bash +x
sudo docker-compose --project-name=${JOB_NAME} stop &> /dev/null || true &> /dev/null
sudo docker-compose --project-name=${JOB_NAME} rm --force &> /dev/null || true &> /dev/null
sudo docker stop `docker ps -a -q -f status=exited` &> /dev/null || true &> /dev/null
sudo docker rm -v `docker ps -a -q -f status=exited` &> /dev/null || true &> /dev/null
sudo docker rmi `docker images --filter \'dangling=true\' -q --no-trunc` &> /dev/null || true &> /dev/null
'''
      }
    }
  }
}