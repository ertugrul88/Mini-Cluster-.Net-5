## Mini Cluster .Net 5 sample-app 
![Alt text](https://github.com/ertugrul88/sample-app/raw/master/diagram.jpeg?raw=true "Mini Cluster")

## 1- Git clone repository
```
ertugrul@ubuntu1:~/case$ git clone your-git-url
```
## 2- Create DockerFile
```
ertugrul@ubuntu1:~/case/sample-app$ vim DockerFile
```
## 3- Edit and Save DockerFile
```
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["sample-app.csproj", "."]
RUN dotnet restore "./sample-app.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "sample-app.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "sample-app.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "sample-app.dll"]
```
## 4- Create Docker Image
```
ertugrul@ubuntu1:~/case/sample-app$ docker build -t ertugrul88/sampleapp -f Dockerfile .
```
## 5- Check Docker Container
```
ertugrul@ubuntu1:~/case/sample-app$ docker run -p 5000:80 ertugrul88/sampleapp 
Go to local link your browser >> http://localhost:5000/WeatherForecast
```
## 6- Authentication hub.docker.com and push image
```
ertugrul@ubuntu1:~/case/sample-app$ docker push ertugrul88/sampleapp:latest
```
## 7- Create pod.yaml file
```
apiVersion: v1
kind: Pod
metadata:
  name: sample-app-pod
  labels:
    app: sample-app
spec:
   containers:
      - name: sample-app
        image: ertugrul88/sampleapp:latest
        imagePullPolicy: Never
        ports:
         - containerPort: 3000
        resources:
         limits:
          cpu: "1"
          memory: 512Mi
         requests:
          cpu: "0.2"
          memory: 256Mi
 ```
## 8- Create deployment.yaml file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
    app: sample-deployment
    tier: cloud
spec:
 replicas: 2
 template:
  metadata:
   name: sample-app-pod
   labels:
    app: sample-app
  spec:
   containers:
    - name: sample-app
      image: ertugrul88/sampleapp:latest
      imagePullPolicy: Never
      ports:
       - containerPort: 3000
      resources:
       limits:
        cpu: "1"
        memory: 512Mi
       requests:
        cpu: "0.2"
        memory: 256Mi
 selector:
  matchLabels:
   app: sample-app
```

## 9- kubernetes deploy app
```
ertugrul@ubuntu1:~/case/sample-app$ kubectl apply -f deployment.yaml
```
## 10- Create service.yaml file
```
apiVersion: v1
kind: Service
metadata:
  name: sample-app-copy-service
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30009
  selector:
      app: sample-app
```

## 11- kubernetes deploy service
```
ertugrul@ubuntu1:~/case/sample-app$ kubectl apply -f service.yaml
```

## 12-  minikube enable ingress
```
ertugrul@ubuntu1:~/case/sample-app$ minikube addons enable ingress
```
## 13- Create ingress.yaml file
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-app-ingress
  labels:
    name: sample-app-ingress
spec:
  rules:
  - host: weatherforecast-sample-app.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: sample-deployment
            port:
              number: 3000
```

## 14- kubernetes deploy ingress
```
 ertugrul@ubuntu1:~/case/sample-app$ kubectl apply -f ingress.yaml
 ```
## 15- Edit /etc/hosts add app local domain adress
```
your-node-ip  weatherforecast-sample-app.com
```
## 16- Finish
```
Go to local link your browser >> http://weatherforecast-sample-app.com/WeatherForecast
```
