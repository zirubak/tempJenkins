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
          // IT HAS TO USE AFTER THE TXT FILES ARE CREATED !!!!!! //
          //////////  Extract data from *.txt //////////////////////
          // Extract build version string from system_version.txt
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
          ////////////////////////////////////////////////////////////
        }

      }
    }
  }
  environment {
    all_resut = 'FAILURE'
    nameStep = ''
    nameStepFail = ''
    BUILD_VER = ''
    CONTRIBUTORS = ''
    COMMITS = ''
  }
}