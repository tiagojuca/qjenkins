// Necessario aprovar as seguintes assinaturas em
// Jenkins -> Gerenciar -> Seguranca -> ScriptApproval
// method hudson.model.Run getCauses
// method java.time.ZonedDateTime getDayOfWeek
// method org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper getRawBuild
// staticMethod java.time.ZoneId of java.lang.String
// staticMethod java.time.ZonedDateTime now java.time.ZoneId

// Parametros de entrada que nao precisam ser configurados via UI, mudam raramente
def SVN_BASE_URL = "https://svn.altoqi.com.br/svn/Eberick/"
def CHECKOUT_FOLDER_PREFIX = "eb"
def SCRIPTS_FOLDER = "Scripts"
def APP_7ZA_REL_PATH = "${SCRIPTS_FOLDER}\\Utils\\7za.exe"
def PYTHON_COMMAND = "python"
def PYTHON_BUILD_SCRIPT = "build.py"
def CMAKE_BINARY_DIR = "exeobj_cmake\\VC"
def OWL_VERSION = "6.42"
def BUNDLE_OUT_DIR = "D:\\jenkins\\jobs\\${env.JOB_NAME}\\builds\\${env.BUILD_NUMBER}"
def BUNDLE_PASSWD = "AltoQi12"

// Configuracao do job, vem dos parametros da UI (se manual) ou de um perfil pre-definido
def JOB_PROFILE = ""
def CONFIG_SVN_REVISION = ""
def CONFIG_SVN_BRANCH = ""
def CONFIG_BUILD_TYPE = ""
def CONFIG_TRANSLATION = ""
def CONFIG_PROTECTION = ""
def CONFIG_COTIRE = false
def CONFIG_VLD = false
def CONFIG_PDB = false

// Variaveis de saida populadas nas etapas de configuracao e espelhamento do Pipeline
def SVN_URL = ""
def SVN_REVISION = ""
def CHECKOUT_FOLDER = ""
def CHECKOUT_PATH = ""
def SCRIPTS_PATH = ""
def APP_7ZA_PATH = ""
def CMAKE_PROJECT_FOLDER = ""
def CMAKE_INSTALL_FOLDER = ""
def CMAKE_INSTALL_PREFIX = ""
def CMAKE_PDB_OUTPUT_DIRECTORY = ""
def PYTHON_BUILD_SCRIPT_FLAGS = ""
def BUNDLE_BASENAME = ""
def BUNDLE_NAME_EXEOBJ = ""
def BUNDLE_NAME_PDBS = ""
def BUNDLE_PATH_EXEOBJ = ""
def BUNDLE_PATH_PDBS = ""

pipeline {
  agent any

  triggers {
    // MINUTO HORA DIA_MES MES DIA_SEMANA

    pollSCM('H/5 * * * *')

    cron('''
      20 3 * * *
      30 3 * * 0
    ''')
  }

  parameters {
    string(name: "REVISAO_SVN", defaultValue: "HEAD", description: "Revisao do SVN a ser compilada")
    string(name: "BRANCH_SVN", defaultValue: "trunk", description: "Branch do SVN a ser compilada")
    choice(name: "TIPO_BUILD", choices: ["RELEASE", "DEBUG"], description: "Tipo de construcao")
    choice(name: "TRADUCAO", choices: ["DESABILITADA", "PT-BR", "ES-MX", "EN-US"], description: "Suporte a traducao")
    choice(name: "PROTECAO", choices: ["OAUTH", "RMS", "DEMO", "SL"], description: "Tipo de protecao")
    booleanParam(name: "COTIRE", defaultValue: true, description: "Compilar com Unity Build?")
    booleanParam(name: "VLD", defaultValue: false, description: "Compilar com Visual Leak Detector?")
    booleanParam(name: "PDB", defaultValue: false, description: "Distribuir simbolos de depuracao MSVC?")
  }

  environment {
    SERVER_PROTECAO = "http://aq000043:8080/"
  }

  stages {

    stage('Configurar') {
      steps {

        // Detecta o gatilho para definir o perfil de configuracao do job
        script {
          def gatilho = currentBuild.getBuildCauses()

          def ehManual = gatilho.any {
            it?._class?.contains('UserIdCause')
          }

          def ehSCM = gatilho.any {
            it?._class?.contains('SCMTriggerCause')
          }

          def ehTimer = gatilho.any {
            it?._class?.contains('TimerTriggerCause')
          }

          if (ehManual) {
            JOB_PROFILE = "SOB_DEMANDA"
          } else if (ehSCM) {
            JOB_PROFILE = "RELEASE_CLOUD"
          } else if (ehTimer) {
            def hoje = java.time.ZonedDateTime.now(java.time.ZoneId.of("America/Sao_Paulo"))

            if (hoje.dayOfWeek == java.time.DayOfWeek.SUNDAY) {
              JOB_PROFILE = "RELEASE_VLD_CLOUD"
            }
            else {
              JOB_PROFILE = "DEBUG_CLOUD"
            }
          }
        }

        // Define as configuracoes do job com base nos parametros ou no perfil pre-definido
        script {
          if (JOB_PROFILE == "SOB_DEMANDA") {
            CONFIG_SVN_REVISION = params.REVISAO_SVN
            CONFIG_SVN_BRANCH = params.BRANCH_SVN
            CONFIG_BUILD_TYPE = params.TIPO_BUILD
            CONFIG_TRANSLATION = params.TRADUCAO
            CONFIG_PROTECTION = params.PROTECAO
            CONFIG_COTIRE = params.COTIRE
            CONFIG_VLD = params.VLD
            CONFIG_PDB = params.PDB
          } else if (JOB_PROFILE == "RELEASE_CLOUD") {
            CONFIG_SVN_REVISION = "HEAD"
            CONFIG_SVN_BRANCH = "trunk"
            CONFIG_BUILD_TYPE = "RELEASE"
            CONFIG_TRANSLATION = "DESABILITADA"
            CONFIG_PROTECTION = "OAUTH"
            CONFIG_COTIRE = true
            CONFIG_VLD = false
            CONFIG_PDB = false
          } else if (JOB_PROFILE == "DEBUG_CLOUD") {
            CONFIG_SVN_REVISION = "HEAD"
            CONFIG_SVN_BRANCH = "trunk"
            CONFIG_BUILD_TYPE = "DEBUG"
            CONFIG_TRANSLATION = "DESABILITADA"
            CONFIG_PROTECTION = "OAUTH"
            CONFIG_COTIRE = true
            CONFIG_VLD = false
            CONFIG_PDB = false
          } else if (JOB_PROFILE == "RELEASE_ESMX_CLOUD") {
            CONFIG_SVN_REVISION = "HEAD"
            CONFIG_SVN_BRANCH = "trunk"
            CONFIG_BUILD_TYPE = "RELEASE"
            CONFIG_TRANSLATION = "ES-MX"
            CONFIG_PROTECTION = "OAUTH"
            CONFIG_COTIRE = true
            CONFIG_VLD = false
            CONFIG_PDB = false
          } else if (JOB_PROFILE == "RELEASE_VLD_CLOUD") {
            CONFIG_SVN_REVISION = "HEAD"
            CONFIG_SVN_BRANCH = "trunk"
            CONFIG_BUILD_TYPE = "RELEASE"
            CONFIG_TRANSLATION = "DESABILITADA"
            CONFIG_PROTECTION = "OAUTH"
            CONFIG_COTIRE = true
            CONFIG_VLD = true
            CONFIG_PDB = false
          }
        }

        // Popula dados do workspace com base nos parametros de SVN
        script {
          def rota = CONFIG_SVN_BRANCH == "trunk" ? "trunk" : "branches/${CONFIG_SVN_BRANCH}"
          SVN_URL = "${SVN_BASE_URL}${rota}"

          CHECKOUT_FOLDER = "${CHECKOUT_FOLDER_PREFIX}_${CONFIG_SVN_BRANCH}"
          CHECKOUT_PATH = "${env.WORKSPACE}\\${CHECKOUT_FOLDER}"
          SCRIPTS_PATH = "${CHECKOUT_PATH}\\${SCRIPTS_FOLDER}"
          APP_7ZA_PATH = "${CHECKOUT_PATH}\\${APP_7ZA_REL_PATH}"
        }

        // Gera o comando de build com base nas opcoes selecionadas
        script {
          if (CONFIG_BUILD_TYPE == "DEBUG") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-B=deb"
          } else {
            PYTHON_BUILD_SCRIPT_FLAGS += "-B=rel"
          }

          PYTHON_BUILD_SCRIPT_FLAGS += " -V=22 -TS=143"

          PYTHON_BUILD_SCRIPT_FLAGS += " "
          if (CONFIG_PROTECTION == "SL") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-SL"
          } else if (CONFIG_PROTECTION == "DEMO") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-D"
          } else if (CONFIG_PROTECTION == "RMS") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-R"
          } else {
            PYTHON_BUILD_SCRIPT_FLAGS += "-O"
          }

          if (CONFIG_TRANSLATION != "DESABILITADA") {
            PYTHON_BUILD_SCRIPT_FLAGS += " -T -Lang="
            if (CONFIG_TRANSLATION == "EN-US") {
              PYTHON_BUILD_SCRIPT_FLAGS += "en-US"
            } else if (CONFIG_TRANSLATION == "ES-MX") {
              PYTHON_BUILD_SCRIPT_FLAGS += "es-MX"
            } else {
              PYTHON_BUILD_SCRIPT_FLAGS += "pt-BR"
            }
          }

          if (CONFIG_COTIRE) {
            PYTHON_BUILD_SCRIPT_FLAGS += " -C"
          }

          if (CONFIG_VLD) {
            PYTHON_BUILD_SCRIPT_FLAGS += " -L"
          }

          PYTHON_BUILD_SCRIPT_FLAGS += " -P2"
        }

        // Monta nome da pasta a ser gerada pelo comando de build
        script {
          if (CONFIG_BUILD_TYPE == "DEBUG") {
            CMAKE_PROJECT_FOLDER += "deb"
          } else {
            CMAKE_PROJECT_FOLDER += "rel"
          }

          CMAKE_PROJECT_FOLDER += "_vs22_msvc143"

          if (CONFIG_PROTECTION == "SL") {
            CMAKE_PROJECT_FOLDER += "_sl"
          } else if (CONFIG_PROTECTION == "DEMO") {
            CMAKE_PROJECT_FOLDER += "_demo"
          } else if (CONFIG_PROTECTION == "RMS") {
            CMAKE_PROJECT_FOLDER += "_rms"
          } else {
            CMAKE_PROJECT_FOLDER += "_oauth"
          }

          if (CONFIG_TRANSLATION != "DESABILITADA") {
            CMAKE_PROJECT_FOLDER += "_tr"
          }

          if (CONFIG_COTIRE) {
            CMAKE_PROJECT_FOLDER += "_cotire"
          }

          if (CONFIG_VLD) {
            CMAKE_PROJECT_FOLDER += "_vld"
          }
        }

        // Monta nome da pasta do comando de instalacao do CMake
        script {
          if (CONFIG_BUILD_TYPE == "DEBUG") {
            CMAKE_INSTALL_FOLDER += "Db"
          } else {
            CMAKE_INSTALL_FOLDER += "Rel"
          }

          CMAKE_INSTALL_FOLDER += "_OWL${OWL_VERSION}"

          if (CONFIG_VLD) {
            CMAKE_INSTALL_FOLDER += "_VLD"
          }

          if (CONFIG_PROTECTION != "DEMO") {
            if (CONFIG_PROTECTION == "SL") {
              CMAKE_INSTALL_FOLDER += "_SL"
            } else if (CONFIG_PROTECTION == "RMS") {
              CMAKE_INSTALL_FOLDER += "_RMS"
            } else {
              CMAKE_INSTALL_FOLDER += "_OAUTH"
            }
          }

          CMAKE_INSTALL_FOLDER += "_x64"

          if (CONFIG_PROTECTION == "DEMO") {
            CMAKE_INSTALL_FOLDER += "_Demo"
          }

          if (CONFIG_COTIRE) {
            CMAKE_INSTALL_FOLDER += "_COTIRE"
          }

          if (CONFIG_TRANSLATION != "DESABILITADA") {
            CMAKE_INSTALL_FOLDER += "_TRA"
          }

          CMAKE_INSTALL_PREFIX = "${CHECKOUT_PATH}\\${CMAKE_BINARY_DIR}\\${CMAKE_INSTALL_FOLDER}"
          CMAKE_PDB_OUTPUT_DIRECTORY = "${CMAKE_INSTALL_PREFIX}_pdb"

          env.EXEOBJ = "${CMAKE_INSTALL_PREFIX}"
        }

        echo CMAKE_INSTALL_PREFIX;

      }
    }

    stage("Espelhar") {
      steps {

        script {
          bat "svn checkout -r ${CONFIG_SVN_REVISION} ${SVN_URL} ${CHECKOUT_FOLDER}"

          if (CONFIG_SVN_REVISION != "HEAD") {
            SVN_REVISION = CONFIG_SVN_REVISION
          } else {
            SVN_REVISION = bat(
              script: """
                @echo off
                for /f "tokens=2" %%i in ('svn info "${CHECKOUT_PATH}" ^| findstr /R "^Revision:"') do @echo %%i
              """,
              returnStdout: true
            ).trim()
          }
        }

        script {
          BUNDLE_BASENAME += "${SVN_REVISION}"

          if (CONFIG_SVN_BRANCH != "trunk") {
            BUNDLE_BASENAME += "_${CONFIG_SVN_BRANCH}"
          }

          if (CONFIG_BUILD_TYPE == "DEBUG") {
            BUNDLE_BASENAME += "_Debug"
          }

          if (CONFIG_VLD) {
            BUNDLE_BASENAME += "_VLD"
          }

          if (CONFIG_TRANSLATION != "DESABILITADA") {
            BUNDLE_BASENAME += "_Traducao"
          }

          if (CONFIG_PROTECTION == "SL") {
            BUNDLE_BASENAME += "_SL"
          } else if (CONFIG_PROTECTION == "DEMO") {
            BUNDLE_BASENAME += "_Demo"
          } else if (CONFIG_PROTECTION == "RMS") {
            BUNDLE_BASENAME += "_RMS"
          } else {
            BUNDLE_BASENAME += "_Cloud"
          }

          BUNDLE_NAME_EXEOBJ = "exeobj_${BUNDLE_BASENAME}.zip"
          BUNDLE_NAME_PDBS = "pdbs_${BUNDLE_BASENAME}.zip"

          BUNDLE_PATH_EXEOBJ = "${BUNDLE_OUT_DIR}\\${BUNDLE_NAME_EXEOBJ}"
          BUNDLE_PATH_PDBS = "${BUNDLE_OUT_DIR}\\${BUNDLE_NAME_PDBS}"

          currentBuild.displayName = "Eberick_${BUNDLE_BASENAME}"
        }

      }
    }

    // stage("Analisar") {

    //   steps {
    //     echo "Analise Encoding"
    //   }

    //   steps {
    //     echo "Analise Estatica"
    //   }

    //   steps {
    //     echo "Analise Resources"
    //   }

    //   steps {
    //     echo "Analise Coverage"
    //   }

    //   script {
    //     if (CONFIG_COTIRE) {
    //       echo "Analise Cpps Isolados"
    //     }
    //   }

    // }

    stage("Construir") {
      steps {
        dir(SCRIPTS_PATH) {
          bat "${PYTHON_COMMAND} ${PYTHON_BUILD_SCRIPT} ${PYTHON_BUILD_SCRIPT_FLAGS}"
        }
      }
    }

    stage("Envelopar") {
      steps {
        dir(SCRIPTS_PATH) {
          bat "${SCRIPTS_PATH}\\ProtegeEberick.bat ENVELOPE_THEMIDA HASH"
          bat "type ${CMAKE_INSTALL_PREFIX}\\Eberick_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\Eberick_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\EberickAdmin_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\SecureEngineSDK64.dll"
        }
      }
    }

    stage("Distribuir") {
      steps {
        script {
          def cmd = "${APP_7ZA_PATH} a -r -tzip -p${BUNDLE_PASSWD}"

          bat "${cmd} ${BUNDLE_PATH_EXEOBJ} ${CMAKE_INSTALL_PREFIX}\\*"
          if (CONFIG_PDB) {
            bat "${cmd} ${BUNDLE_PATH_PDBS} ${CMAKE_PDB_OUTPUT_DIRECTORY}\\*"
          }
        }
      }
    }

    // stage("Testar") {
    //   steps {
    //     echo "Testando"
    //   }
    // }

  }
}
