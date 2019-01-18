pipeline {
    agent none
    stages {
        stage('Prepare') {
            agent {
                label "worker"
            }
            steps {
                //cleanWs()
                dir('kicad-src') {
                    git 'https://git.launchpad.net/kicad'
                }
                echo 'NICK DEBUG 1'
                sh 'pwd'
                echo 'NICK DEBUG 2'
                sh 'ls ${WORKSPACE}/kicad-src'
            }
        }
        stage('Stash repo') {
            agent {
                label "worker"
            }
            steps {
                sh 'ls'
                stash includes: '**', name: 'kicad-source-repo', useDefaultExcludes: false
            }
        }
        stage('Run Builds') {
            parallel {
                stage('Quick build') {
                    agent {
                        label "generic-alpine"
                    }
                    steps {
                        unstash 'kicad-source-repo'
                        sh 'ls'
                        sh '''#!/bin/bash
                            set -e
                            cd kicad-src
                            mkdir build
                            cd build
                            cmake .. -DCMAKE_BUILD_TYPE=Debug -DBUILD_GITHUB_PLUGIN=ON -DKICAD_SCRIPTING=OFF -DKICAD_SCRIPTING_MODULES=OFF -DKICAD_SCRIPTING_ACTION_MENU=OFF -DKICAD_SCRIPTING_WXPYTHON=OFF -DKICAD_USE_OCE=OFF -DKICAD_SPICE=OFF
                            #make -j4
                        '''
                    }
                    post {
                        always {
                            sh 'echo nickersej'
                        }
                    }
                }
                stage('Clean build') {
                    agent {
                        label "docker_appimage_debug"
                    }
                    steps {
                        unstash 'kicad-source-repo'
                    }
                    post {
                        always {
                            //junit "**/TEST-*.xml"
                            sh 'echo fisk'
                        }
                    }
                }
                stage('Appimage build') {
                    agent {
                        label "master"
                    }
                    steps {
                        echo 'Hello World from appimage build step'
                        //build 'linux-kicad-appimage'
                    }
                    post {
                        always {
                            //junit "**/TEST-*.xml"
                            sh 'echo fisk2'
                        }
                    }
                }
                stage('Archlinux build') {
                    agent {
                        label "docker_generic-archlinux"
                    }
                    steps {
                        git https://aur.archlinux.org/kicad-git.git
                        echo 'Hello World from appimage build step'
                        sh 'makepkg'
                        stash includes: '*pkg.tar.xz', name: 'kicad-aur-git', useDefaultExcludes: false 
                    }
                    post {
                        always {
                            //junit "**/TEST-*.xml"
                            sh 'echo fisk2'
                        }
                    }
                }
                stage('Doxygen build') {
                    agent {
                        label "generic-alpine"
                    }
                    steps {
                        unstash 'kicad-source-repo'
                        dir('kicad-src') {
                            sh 'pwd && ls'
                            sh 'mkdir build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Debug -DBUILD_GITHUB_PLUGIN=ON -DKICAD_SCRIPTING=OFF -DKICAD_SCRIPTING_MODULES=OFF -DKICAD_SCRIPTING_ACTION_MENU=OFF -DKICAD_SCRIPTING_WXPYTHON=OFF -DKICAD_USE_OCE=OFF -DKICAD_SPICE=OFF'
                            sh 'pwd'
                            sh 'cd build && make doxygen-docs -j3'
                            //sh 'cd build && make doxygen-python -j3' // we probably need to enable scripting as well
                        }
                    }
                    post {
                        success {
                            stash includes: 'kicad-src/Documentation/doxygen/html/**', name: 'doxygen', useDefaultExcludes: false
                            //stash includes: 'kicad-src/build/pcbnew/doxygen-python/**', name: 'doxygen-python', useDefaultExcludes: false
                        }
                    }
                }
            }
        }
        stage('Share') {
            agent {
                label "worker"
            }
            steps {
                //unstash 'test'
                sh 'ls'
                //sh 's3cmd put build-cmake/kicad-doc-HEAD.tar.gz s3://kicad-downloads/docs/'

                sh 's3cmd ls s3://kicad-downloads/appimage/'
                copyArtifacts fingerprintArtifacts: true, projectName: 'linux-kicad-appimage', selector: lastSuccessful()
                sh 's3cmd put KiCad*.AppImage s3://kicad-downloads/appimage/testing/'
                sh 's3cmd ls s3://kicad-downloads/appimage/'

                unstash 'doxygen'
                //unstash 'doxygen-python'
                sh 'pwd'
                sh 'ls'
                sh 's3cmd put --guess-mime-type --recursive kicad-src/Documentation/doxygen/html/* s3://kicad-downloads/doxygen/'
                sh 's3cmd put --mime-type=text/css kicad-src/Documentation/doxygen/html/tabs.css s3://kicad-downloads/doxygen/'

                unstash 'kicad-aur-git'
                sh 's3cmd put *pkg.tar.xz s3://kicad-downloads/archlinux/testing/'
            }
        }  
    }
}



