pipeline {
  agent none
  stages {
    stage('tag') {
      agent {
        node {
          label 'nightly-oe-classic'
        }

      }
      environment {
        nameStep = 'tag'
      }
      steps {
        deleteDir()
        dir(path: 'manifest-update') {
          git(url: 'http://github.trellisware.com/software-team/manifest-update', branch: 'stable-6.0', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
        }

        dir(path: 'openembedded-dev-trellisware') {
          git(url: 'http://github.trellisware.com/software-team/openembedded-dev-trellisware', branch: 'stable-6.0-tsm-e', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
        }

        dir(path: 'linux-manifest') {
          git(url: 'http://github.trellisware.com/software-team/linux-manifest', branch: 'stable-6.0-tsm-x', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
        }

        dir(path: 'automation') {
          git(url: 'http://github.trellisware.com/software-team/automation', branch: 'master', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
        }

        dir(path: 'openembedded-dev-trellisware') {
          sh '''set -x
# checks if tag is empty and if so finds latest tag and increments it
LATEST_TAG=`git describe --abbrev=0`
if [ -z "$omap" ]
then
    LAST=${LATEST_TAG: -4}
    RC_NUM="${LAST//[!0-9]/}"
    NEXT_RC_NUM="$(($RC_NUM + 1))"
    omap=$(cut -f1 -d"c" <<< $LATEST_TAG)c$NEXT_RC_NUM
fi

# checks for git version tag existence, makes one if tag dne
git ls-remote --tags origin $omap > tag.txt

echo \'Please check your changes\' > ../manifest-update/contributors.txt
echo \'tsm-e\' >> ../manifest-update/contributors.txt
echo \'tsm-e \'$omap > ../manifest-update/release-notes.txt

if [ -s tag.txt ]
then
    echo "I have a tag"

    LAST_TAG=`git describe --abbrev=0 ${omap}^`
    git log --format=\'%ae\' $LAST_TAG..$omap | sort -u >> ../manifest-update/contributors.txt
    git log $LAST_TAG..$omap --pretty=format:%s%n%b --first-parent >> ../manifest-update/release-notes.txt
else
    echo "I have no tag"

    LAST_TAG=`git describe --abbrev=0`
    git log --format=\'%ae\' $LAST_TAG..HEAD | sort -u >> ../manifest-update/contributors.txt
    git log $LAST_TAG..HEAD --pretty=format:%s%n%b --first-parent >> ../manifest-update/release-notes.txt

    git tag -a $omap --file=../manifest-update/release-notes.txt
    git push origin $omap
fi'''
        }

        dir(path: 'linux-manifest') {
          sh '''set -x
# checks if tag is empty and if so finds latest tag and increments it
LATEST_TAG=`git describe --abbrev=0`
if [ -z "$imx6" ]
then
    LAST=${LATEST_TAG: -4}
    RC_NUM="${LAST//[!0-9]/}"
    NEXT_RC_NUM="$(($RC_NUM + 1))"
    imx6=$(cut -f1 -d"c" <<< $LATEST_TAG)c$NEXT_RC_NUM
fi

# checks for git version tag existence, makes one if tag dne
git ls-remote --tags origin $imx6 > tag.txt

echo \'tsm-x\' >> ../manifest-update/contributors.txt
echo \'tsm-x \'$imx6 >> ../manifest-update/release-notes.txt

if [ -s tag.txt ]
then
    echo "I have a tag"

    LAST_TAG=`git describe --abbrev=0 ${imx6}^`
    ./log.py -q $LAST_TAG $imx6 -- --format=\'%ae\' | sort -u >> ../manifest-update/contributors.txt
    ./log.py -q $LAST_TAG $imx6 -- --format=\'%s%n%b\' >> ../manifest-update/release-notes.txt
else
    echo "I have no tag"

    LAST_TAG=`git describe --abbrev=0`
    ./log.py -q $LAST_TAG HEAD -- --format=\'%ae\' | sort -u >> ../manifest-update/contributors.txt
    ./log.py -q $LAST_TAG HEAD -- --format=\'%s%n%b\' >> ../manifest-update/release-notes.txt

    # This temporary file is created and deleted for the tagging only
    echo \'tsm-x \'$imx6 > ../manifest-update/imx6onlynotes.txt
    ./log.py -q $LAST_TAG HEAD -- --format=\'%s%n%b\' >> ../manifest-update/imx6onlynotes.txt
    git tag -a $imx6 --file=../manifest-update/imx6onlynotes.txt
    rm -rf ../manifest-update/imx6onlynotes.txt
    git push origin $imx6
fi'''
        }

        dir(path: 'automation') {
          sh '''set -x
./format.py'''
        }

        dir(path: 'manifest-update') {
          sh '''set -x
cat notes.txt
cat contributors.txt
mv notes.txt ..
mv contributors.txt ..'''
        }

        script {
          BUILD_VER = sh(script: "find ${WORKSPACE} -name system_version.txt|xargs cat",
          returnStdout: true).trim()
          BUILD_VER += "-b"
          BUILD_VER += sh(script: "find ${WORKSPACE} -name system_build.txt|xargs cat",
          returnStdout: true).trim()
          echo "## BUILD_VER: ${BUILD_VER}"
          // Extract list of contributors from contributors.txt
          CONTRIBUTORS = sh(script: "find ${WORKSPACE} -name contributors.txt|xargs cat",
          returnStdout: true)
          echo "## CONTRIBUTORS: ${CONTRIBUTORS}"
          // Extract list of commit from note.txt
          COMMITS = sh(script: "find ${WORKSPACE} -name note.txt|xargs cat",
          returnStdout: true)
          echo "## COMMITS: ${COMMITS}"
        }

      }
    }
    stage('builds') {
      parallel {
        stage('build tsm-x') {
          agent {
            node {
              label 'nightly-yocto'
            }

          }
          environment {
            nameStep = 'build tsm-x'
          }
          steps {
            deleteDir()
            dir(path: 'linux-build-scripts') {
              git(url: 'http://github.trellisware.com/software-team/linux-build-scripts', branch: 'stable-6.0-tsm-x', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
            }

            dir(path: 'manifest-update') {
              git(url: 'http://github.trellisware.com/software-team/manifest-update', branch: 'stable-6.0', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
            }

            dir(path: 'linux-manifest') {
              git(url: 'http://github.trellisware.com/software-team/linux-manifest', branch: 'stable-6.0-tsm-x', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
            }

            dir(path: 'automation') {
              git(url: 'http://github.trellisware.com/software-team/automation', branch: 'master', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
            }

            dir(path: 'linux-manifest') {
              withCredentials(bindings: [[
                                                                                                                        $class: 'UsernamePasswordMultiBinding',
                                                                                                                        credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744',
                                                                                                                        usernameVariable: 'jenkins_build',
                                                                                                                        passwordVariable: 'ppr0jm11eRdjV900gZCk'
                                                                                                                      ]]) {
                  sh '''set -x
if [ -z "$imx6" ]
then
    imx6=`git describe --abbrev=0`
fi'''
                  sh 'git ls-remote --exit-code --tags origin $imx6'
                }

              }

              dir(path: 'automation') {
                sh '''// checks for svn linux distro version tag existence, builds if tag dne
set -x
echo $PATH
rm -rf ../builds
if ! svn ls https://svn.trellisware.com/twt-release/restricted/shadow/device/components/gpp/linux/${imx6} &> /dev/null; then
    echo "The svn tag ${imx6} does not exist and now we will attempt to build it"
    cbe bash -s <<EOF
    set -x
    set -e
    if ${svn}; then
        echo "svn is true"
        ./linux-release-tsm-x.sh ${imx6} /home/*/ci/workspace/packager stable-6.0
    else
        echo "svn is false"
        ./linux-release-tsm-x.sh --dry-run ${imx6} /home/*/ci/workspace/packager stable-6.0
    fi
EOF
fi'''
              }

            }
          }
          stage('build tsm-e') {
            agent {
              node {
                label 'nightly-oe-classic'
              }

            }
            environment {
              nameStep = 'build tsm-e'
            }
            steps {
              dir(path: 'linux-build-scripts') {
                git(url: 'http://github.trellisware.com/software-team/linux-build-scripts', branch: 'stable-6.0-tsm-e', credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744')
              }

              dir(path: 'openembedded-dev-trellisware') {
                withCredentials(bindings: [[
                                                                                                                                    $class: 'UsernamePasswordMultiBinding',
                                                                                                                                    credentialsId: '7375b363-bbe2-4ce3-a6fb-14926ba42744',
                                                                                                                                    usernameVariable: 'jenkins_build',
                                                                                                                                    passwordVariable: 'ppr0jm11eRdjV900gZCk'
                                                                                                                  ]]) {
                    sh 'git config --global user.email "jenkins_build@trellisware.com"'
                    sh 'git config --global user.name "Jenkins Build"'
                    sh '''// checks for git version tag existence, exits if tag dne
git ls-remote --exit-code --tags origin $omap'''
                  }

                }

                dir(path: 'automation') {
                  sh '''// checks for svn linux distro version tag existence, builds if tag dne

set -x
rm -rf ../builds
echo $omap
if ! svn ls https://svn.trellisware.com/twt-release/restricted/ghostii/device/components/slice/gpp/operational/${omap} &> /dev/null; then
    echo "The svn tag ${omap} does not exist and now we will attempt to build it"
    cbe bash -s <<EOF
    set -x
    set -e
    if ${svn}; then
        echo "svn is true"
        ./linux-release-tsm-e.sh ${omap} /home/*/ci/workspace/packager stable-6.0
    else
        echo "svn is false"
        ./linux-release-tsm-e.sh --dry-run ${omap} /home/*/ci/workspace/packager stable-6.0
    fi
EOF
fi'''
                }

              }
            }
          }
        }
        stage('package') {
          agent {
            node {
              label 'nightly-oe-classic'
            }

          }
          environment {
            nameStep = 'package'
          }
          steps {
            dir(path: 'automation') {
              sh '''set -x
if [ -z "$imx6" ]
then
    cd ../linux-manifest
    imx6=`git describe --abbrev=0`
    cd ../automation
fi
if [ -z "$omap" ]
then
    cd ../openembedded-dev-trellisware
    omap=`git describe --abbrev=0`
    cd ../automation
fi
cbe ./update.sh ${imx6} ${omap} ${BUILD_NUMBER} stable-6.0 ../manifest-update false ${release} ${push} true'''
            }

            dir(path: 'manifest-update') {
              archiveArtifacts '**'
            }

          }
        }
        stage('test') {
          agent {
            node {
              label 'nightly-oe-classic'
            }

          }
          environment {
            nameStep = 'test'
          }
          steps {
            build(job: 'test-packager', propagate: true)
          }
        }
      }

      post{
        always{
        
        }
        failure{
        
        }
      }
      
      environment {
        all_resut = 'FAILURE'
        nameStep = ''
        nameStepFail = ''
        BUILD_VER = ''
        CONTRIBUTORS = ''
        nameStepCOMMITS = ''
      }
    }
