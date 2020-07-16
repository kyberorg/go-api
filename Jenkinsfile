@Library('common-lib@1.4') _
pipeline {
    agent any;
    stages {
        stage('Docker') {
            steps {
                script {
                    def repo = 'kyberorg/go-api';
                    def tags = [];
                    String tag;
                    if (env.BRANCH_NAME.equals("master")) {
                        tag = "stable";
                    } else {
                        tag = env.BRANCH_NAME;
                    }
                    tags << tag;

                    dockerBuild(repo: repo, tags: tags);
                    dockerLogin(creds: 'hub-docker');
                    dockerPush();
                    dockerLogout();
                    dockerClean();
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    String hookUrl;
                    switch (env.BRANCH_NAME) {
                        case "master":
                            hookUrl = "?tag=stable";
                            break;
                        default:
                            hookUrl = "?tag=" + env.BRANCH_NAME;
                            break;
                    }
                    //no hook - no deploy
                    if(hookUrl.equals('')) { return; }
                    deployToSwarm(hookUrl: hookUrl);
                    sleep(10); //pause for application to be started
                }
            }
        }
    }
    post {
        always {
            cleanWs();
        }
    }
}
