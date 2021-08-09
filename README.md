[![ci](https://github.com/ahmadarif/docker-laravel/actions/workflows/main.yaml/badge.svg)](https://github.com/ahmadarif/docker-laravel/actions/workflows/main.yaml)

# Description

    Docker image for Laravel/Lumen Apps

## Sample Usage with caching

```Dockerfile
FROM ahmadarif/docker-laravel:latest

WORKDIR /var/www/html
COPY . .
RUN composer install

EXPOSE 80
ENTRYPOINT ["/usr/bin/supervisord"]
```

## Sample usage in gitlab (CI/CD)

```shell
Deploy:
  stage: deploy
  image: ahmadarif/rancher-cli-k8s
  only:
    - develop
    - master
  variables:
    IMAGE_TAG: your-docker-image
    CI_JOB_TIMESTAMP: $(date --utc -Iseconds)
    K8S_ENDPOINT: https://your-rancher-domain.com
    K8S_ACCESS_KEY:
    K8S_SECRET_KEY:
    K8S_PROJECT_ID:
    K8S_NAMESPACE:
    K8S_WORKLOAD:
  before_script:
    # optional for rewrite internal domain
    - echo "127.0.0.1 your-rancher-domain.com" | tee -a /etc/hosts >/dev/null
  script:
    - mkdir ~/.rancher
    - echo "{\"Servers\":{\"rancherDefault\":{\"accessKey\":\"$K8S_ACCESS_KEY\",\"secretKey\":\"$K8S_SECRET_KEY\",\"tokenKey\":\"$K8S_ACCESS_KEY:$K8S_SECRET_KEY\",\"url\":\"$K8S_ENDPOINT\",\"project\":\"$K8S_PROJECT_ID\",\"cacert\":\"\"}},\"CurrentServer\":\"rancherDefault\"}" > ~/.rancher/cli2.json
    - rancher kubectl --insecure-skip-tls-verify --namespace=$K8S_NAMESPACE patch deployment $K8S_WORKLOAD -p "{\"spec\":{\"template\":{\"spec\":{\"containers\":[{\"name\":\"intra-api\",\"image\":\"$IMAGE_TAG\",\"env\":[{\"name\":\"FORCE_RESTART_AT\",\"value\":\"$CI_JOB_TIMESTAMP\"}]}]}}}}"
```
