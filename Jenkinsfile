pipeline {
  agent { label 'sonarqube' }

  environment {
     PROJECT_HOME = "D:\\proj\\Admin1.6"
  }
  options {
    timestamps()
    timeout(time: 120, unit: 'MINUTES')
    ansiColor('xterm')
  }

  stages {
    stage ('Setup project') {
      steps {
        checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], browser: [$class: 'TFS2013GitRepositoryBrowser', repoUrl: 'http://ukcamstfsliveap:8080/tfs/EDSCollection/EDS/Software%20Production/_git/spt-sonarqube-analysis'], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'indexadmdevelop', url: 'ssh://ukcamstfsliveap:22/tfs/EDSCollection/EDS/_git/spt-sonarqube-analysis']]]
        checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], browser: [$class: 'TFS2013GitRepositoryBrowser', repoUrl: 'http://ukcamstfsliveap:8080/tfs/EDSCollection/EDS/Software%20Production/_git/spt-scripts-grok'], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'spt-scripts-grok']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'indexadmdevelop', url: 'ssh://ukcamstfsliveap:22/tfs/EDSCollection/EDS/_git/spt-scripts-grok']]]
        // remove old log files
        bat '@for /r %%l in (*.log) do @del /f /s /q "%%l"'
        // make sure the project home directory is available
        bat '@if not exist "%PROJECT_HOME%" @mkdir "%PROJECT_HOME%"'
        bat '@if not exist "%PROJECT_HOME%\\inc" @mkdir "%PROJECT_HOME%\\inc"'
        // copy required files to project home
        bat '@copy "%WORKSPACE%\\cxx-includes\\.includes" "%PROJECT_HOME%\\.includes" /Y'
        bat '@copy "%WORKSPACE%\\cxx-includes\\default-macros-dll.h" "%PROJECT_HOME%\\inc\\default-macros.h" /Y'
        bat '@copy "%WORKSPACE%\\eng-15.1\\.files" "%PROJECT_HOME%\\.files" /Y'
        bat '@copy "%WORKSPACE%\\eng-15.1\\.subfolders" "%PROJECT_HOME%\\.subfolders" /Y'
        bat '@copy "%WORKSPACE%\\eng-15.1\\.xd" "%PROJECT_HOME%\\.xd" /Y'
        bat '@copy "%WORKSPACE%\\eng-15.1\\macros.h" "%PROJECT_HOME%\\inc\\macros.h" /Y'
        bat '@copy "%WORKSPACE%\\eng-15.1\\sonar-project.properties" "%PROJECT_HOME%\\sonar-project.properties" /Y'
      }
    }
    stage ('SCM') {
        stage ('ClearCase') {
          steps {
            bat '@call "%WORKSPACE%\\spt-scripts-grok\\clearcase\\cc-scm-dynamic.cmd" divya_admin1.6'
            bat '@call "%WORKSPACE%\\spt-scripts-grok\\utils\\copy-source.cmd" M:\\divya_admin1.6'
          }
      }
    }
    stage ('Generate reports') {
        stage ('Admin1.6') {
          steps {
            bat '@call "%WORKSPACE%\\scripts\\generate-cppcheck-reports.cmd" src0\\Admin1.6'
          }
        }
    }
    stage ('Prepare analysis') {
      parallel {
        stage ('C#') {
          steps {
            bat '@call "%WORKSPACE%\\spt-scripts-grok\\utils\\convert-cs-encoding.cmd" UTF8'
          }
        }
        stage ('C-family') {
          steps {
            bat '@call "%WORKSPACE%\\spt-scripts-grok\\utils\\convert-cfamily-encoding.cmd" UTF8'
          }
        }
      }
    }
    stage ('Analysis') {
      steps {
        bat '@call "%WORKSPACE%\\scripts\\sonar-project.cmd"'
      }
    }
  }

  post {
    always {
      archiveArtifacts allowEmptyArchive: true, artifacts: '*.log'
    }
  }
}
