node {
  def project = 'devops-preprod'
  def appName = 'gerua'
  def feSvcName = "${appName}-frontend"
  def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
  sh("docker build -t ${imageTag} .")

  stage 'Run Go tests'
  sh("docker run ${imageTag} go test")

  stage 'Push image to registry'
  sh("gcloud docker -- push ${imageTag}")

  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    // Roll out to canary environment
    case "canary":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gerua:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
        sh("kubectl --namespace=production apply -f k8s/services/")
        sh("kubectl --namespace=production apply -f k8s/canary/")
        sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break

    // Roll out to production
    case "master":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gerua:1.0.0#${imageTag}#' ./k8s/production/*.yaml")
        sh("kubectl --namespace=production apply -f k8s/services/")
        sh("kubectl --namespace=production apply -f k8s/production/")
        sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break
  }
}
