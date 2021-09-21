import java.text.SimpleDateFormat

def props
def label = "jenkins-slave-${UUID.randomUUID().toString()}"
currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-" + env.BUILD_NUMBER


podTemplate(
  label: label,
  namespace: "go-demo-3-build",
  serviceAccount: "build",
  yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm
    image: vfarcic/helm:3.0.2
    command: ["cat"]
    tty: true
    volumneMounts:
    - name: build-config
      mountPath: /etc/config
  - name: kubectl
    image: vfarcic/kubectl
    command: ["cat"]
    tty: true
  - name: golang
    image: golang:1.12
    command: ["cat"]
    tty: true
  volumes:
  - name: build-config
    configMap:
      name: build-config
"""
) {
  node(label) {
    stage("build") {
      container("helm") {
        sh "cp /etc/config/build-config.properties ."
        props = readProperties interpolate: true, file: "build-config.properties"
      }
      node("docker") { // nesting docker node inside pod node so that when things are building on the docker node, the pod nodes dont' die since they are ephemeral
        checkout scm
        k8sBuildImageBeta(props.image)
      }
    }
    stage("func-test") {
      try {
        container("helm") {
          checkout scm
          k8sUpgradeBeta(props.project, props.domain, "--set replicaCount=2 --set dbReplicaCount=1")
        }
        container("kubectl") {
          k8sRolloutBeta(props.domain)
        }
        container("golang") {
          k8sFuncTestGolang(props.project, props.domain)
        }
      } catch(e) {
          error "Failed functional tests"
      } finally {
        container("helm") {
          k8sDeleteBeta(props.project)
        }
      }
    }
    if ("${BRANCH_NAME}" == "master") {
      stage("release") {
        node("docker") {
          k8sPushImage(props.image)
        }
        container("helm") {
          k8sPushHelm(props.project, props.chartVer, props.cmAddr)
        }
      }
      stage("deploy") {
        try {
          container("helm") {
            k8sUpgrade(props.project, props.addr)
          }
          container("kubectl") {
            k8sRollout(props.project)
          }
          container("golang") {
            k8sProdTestGolang(props.addr)
          }
        } catch(e) {
          container("helm") {
            k8sRollback(props.project)
          }
        }
      }
    }
  }
}
