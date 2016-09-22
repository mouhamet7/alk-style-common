#!/usr/bin/env ptyhon
import groovy.transform.Field

@Field def repo_slug = "alk-style-common" as String
@Field def nodejs = "" as String
@Field def utils = "" as String
@Field def git_commit = "" as String

timestamps{
  node('nodejs'){
    try {
      dir("/home/deploy/${repo_slug}"){
        fileLoader.withGit('https://github.com/alkemics/jenkins-lib.git', 'master', 'lucy', '') {
          nodejs = fileLoader.load('Nodejs');
          utils = fileLoader.load('Utils');
        }

        stage('Initialization'){
            // lets enforce branch name as it is a single pipeline
            env.BRANCH_NAME = 'master'
            git_commit = utils.initialize(repo_slug)
        }

        stage ('Install'){
            nodejs.install()
        }

        stage('Publish'){
            nodejs.publish(repo_slug)
        }
        stage("Finalisation"){
            utils.notifyGithub(repo_slug, git_commit, utils.state_success)
        }
      }
    } catch (e) {
      println e
      utils.notifyGithub(repo_slug, git_commit, utils.state_failed)
      if (env.BRANCH_NAME.startsWith('master')){
        utils.notifySlack(utils.buildFailedForSlack(
          "<${env.BUILD_URL}|${repo_slug} ${env.JOB_NAME}#${env.BUILD_NUMBER}>"
        ));
      }
      error 'Build failed'
    }
  }

}
