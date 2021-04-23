# Java & Kubernetes

Construir um ambiente Kubernetes local para que possamos aprender a tecnologia sem medo de errar. Vamos criar os recursos necessários para fazer o deploy no cluster e configurar nossa aplicação a fim de fazer debug enquanto ela está rodando no Kubernetes

## Parte 1 - app básico:

### Requirements:

**Docker & Make (Opcional)**

**Java 16**
### Ferramenas utilizadas:
 + Minikube
 + Kubernetes
 + Docker
 + IDE de sua preferência

### Build e execução:

Aplicação Spring Boot e MySQL rodando em Docker

**Clonar o repositório**
```bash
git clone https://github.com/eduardolm/java-kubernetes.git
```

**Build da aplicação**
```bash
cd java-kubernetes
mvn clean install
```

**Executar o banco de dados em Docker container**
```bash
make run-db
```

**Executar a aplicação**
```bash
java --enable-preview -jar target/java-kubernetes.jar
```

**Testes**

http://localhost:8080/app/users

http://localhost:8080/app/hello

## Parte 2 - Execução do app em Docker:

Criar um Dockerfile:

```yaml
FROM openjdk:16-alpine
RUN mkdir /usr/myapp
COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java --enable-preview $JAVA_OPTS -jar app.jar" ]
```

**Build da aplicação e da imagem Docker**

```bash
make build
```

Executar o banco de dados
```bash
make run-db
```

Executar a aplicação
```bash
make run-app
```

**Testes**

http://localhost:8080/app/users

http://localhost:8080/app/hello

Encerrar a execução:

`
docker stop mysql myapp
`

## Parte 3 - Rodando o app em Kubernetes:

Temos uma aplicação e sua imagem Docker. Agora faremos o deploy da aplicação 
num cluster Kubernetes sendo executado localmente.


Preparação para o deploy

### Executar Minikube
`
make k-setup
`
 Executar minikube, habilitar ingress, metrics-server e criar um namespace no cluster Kubernetes

### Verificar o IP

`
minikube -p dev.to ip
`

### Dashboard Minikube
Interface gráfica, acessada pelo navegador que permite interagir de forma mais simples com o Kuberntes.

`
minikube -p dev.to dashboard
`

### Deploy do banco de dados

Criar o deploy do MySQL e o respectivo serviço

`
make k-deploy-db
`

`
kubectl get pods -n dev-to
`

OU

`
watch k get pods -n dev-to
`


`
kubectl logs -n dev-to -f <pod_name>
`

`
kubectl port-forward -n dev-to <pod_name> 3306:3306
`

## Build e deploy da aplicação

build da aplicação

`
make k-build-app
` 

Criando uma imagem docker dentro da máquina Minikube:

`
make k-build-image
`

OU

`
make k-cache-image
`  

criar o deploy e serviço da aplicação:

`
make k-deploy-app
` 

**Testar**

`
kubectl get services -n dev-to
`

Para acessar a aplicação:

`
minikube -p dev.to service -n dev-to myapp --url
`

Ex:

http://172.17.0.3:32594/app/users
http://172.17.0.3:32594/app/hello

## Varificar os pods

`
kubectl get pods -n dev-to
`

`
kubectl -n dev-to logs myapp-6ccb69fcbc-rqkpx
`

## Mapeando para dev.local

get minikube IP
`
minikube -p dev.to ip
` 

Editando o `hosts` 

`
sudo vim /etc/hosts
`

Replicas
`
kubectl get rs -n dev-to
`

Listando e excluindo pods
`
kubectl get pods -n dev-to
`

`
kubectl delete pod -n dev-to myapp-f6774f497-82w4r
`

Escalando
`
kubectl -n dev-to scale deployment/myapp --replicas=2
`

Testando as répplicas
`
while true
do curl "http://dev.local/app/hello"
echo
sleep 2
done
`
Testando as réplicas e aguardando

`
while true
do curl "http://dev.local/app/wait"
echo
done
`

## Check app url
`minikube -p dev.to service -n dev-to myapp --url`

Modifique seu IP e PORTA conforme necessário

`
curl -X GET http://dev.local/app/users
`

Adicionando um novo usuário
`
curl --location --request POST 'http://dev.local/app/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "new user",
    "birthDate": "2010-10-01"
}'
`

## Parte 4 - depurando a aplicação:

Descomentar `JAVA_OPTS: "-agentlib:jdwp=transport=dt_socket,address=*:5005,server=y,suspend=n"`
no arquivo localizado em `k8s/app/app-configmap.yml`
 
alterar o CMD para o ENTRYPOINT no Dockerfile

`
kubectl get pods -n=dev-to
`

`
kubectl port-forward -n=dev-to <pod_name> 5005:5005
`

## KubeNs e Stern

`
kubens dev-to
`

`
stern myapp
` 

## Executar todos os serviços

`make k:all`


## Referências

https://kubernetes.io/docs/home/

https://minikube.sigs.k8s.io/docs/

## Comandos úteis

```
##List profiles
minikube profile list

kubectl top node

kubectl top pod <nome_do_pod>
```
