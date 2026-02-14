# qjenkins

## ToDo:
Escrever docker-compose para automatizar:

Compilar imagem:
``docker build -t qjenkins:2.550-jdk21 .``

Criar rede compartilhada:
``docker network create jenkins``

Executar container:
``docker run --name qjenkins --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  qjenkins:2.550-jdk21
``
