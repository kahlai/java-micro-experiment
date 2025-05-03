# This repository contains experiment using UBI Micro + JRE runtime only to run quarkus application.

## Introduction
The more things we pack inside a container image and deploy in production environment, the more attacking surface we are exposed to. In most cases, we do not need full JDK to run an Java application. This repository experiment to build a simple JRE only Image and execute a simple quarkus application inside.


## Folder Structure
```
.
├── Dockerfile.base - Docker file to build container image by copy jre into standard UBI micro
├── Dockerfile.jre - Docker file to build sample java app and copy into the above base image.
├── Dockerfile.native - Docker file to build sample quarkus native executable app
├── README.md
└── code-with-quarkus - Folder container sample quarkus REST application
```

## Steps to reproduce the experiment. 

### Build and run quarkus app using Java Runtime Only
1. Download the Java Runtime (JRE) Zip file from Red Hat (e.g https://access.redhat.com/jbossnetwork/restricted/softwareDownload.html?softwareId=107780) or https://adoptium.net/en-GB/download/


2. Unzip tar.xz in to ./jre folder 
e.g 
```
tar -xf OpenJDK21U-jre_x64_linux_hotspot_21.0.7_6.tar.gz
mv jdk-21.0.7+6-jre jre

or 

tar -xf java-21-openjdk-21.0.7.0.6-1.portable.jre.x86_64.tar.xz
mv java-21-openjdk-21.0.7.0.6-1.portable.jre.x86_64 jre
```

1. Build the base image
```
docker build --platform linux/amd64 -t quay.io/kahlai/java-micro:ubi9 -f Dockerfile.base .
```

4. Inspect and see the size of the JRE base images
```
docker images | grep java-micro
```
Example output:
```
quay.io/kahlai/java-micro                             ubi9                 434cd5214844   2 hours ago    188MB
```
Smaller compare to:
```
registry.access.redhat.com/ubi8/openjdk-21            latest               80be32f2c910   2 weeks ago    431MB
```

5. Build the code to test the images with JRE only.
```
docker build --platform linux/amd64 -t quay.io/kahlai/sample-app:ubi9 -f Dockerfile.jre .
```

6. Inspect and see the size of the sample app images
```
docker images | grep sample-app
```
Example output:
```
quay.io/kahlai/sample-app                             ubi9                 f8c65716fd77   15 hours ago    204MB
```

7. Run and Test the simple REST API application
```
docker run --rm --platform linux/amd64 -p 8080:8080 quay.io/kahlai/sample-app:ubi9
```

Example output:
```
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2025-05-02 12:50:46,114 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT on JVM (powered by Quarkus 3.21.3) started in 1.012s. Listening on: http://0.0.0.0:8080
2025-05-02 12:50:46,117 INFO  [io.quarkus] (main) Profile prod activated. 
2025-05-02 12:50:46,117 INFO  [io.quarkus] (main) Installed features: [cdi, rest, smallrye-context-propagation, vertx]
```

7. Test the application (Using new Terminal)
```
curl localhost:8080/hello             
```
Example Output:
```
Hello from Quarkus REST%
```

### Build and run quarkus app using Native Compilation

1. Build the code to test the images with Native Compilation.
```
docker build --platform linux/amd64  -t quay.io/kahlai/sample-app:ubi9-native -f Dockerfile.native .
```


2. Inspect and see the size of the sample app images
```
docker images | grep sample-app
```
Example output:
```
quay.io/kahlai/sample-app                             ubi9-native          9857043f1b01   2 minutes ago   77.4MB
quay.io/kahlai/sample-app                             ubi9                 f8c65716fd77   15 hours ago    204MB
```

3. Run and Test the simple REST API application
```
docker run --rm --platform linux/amd64 -p 8080:8080 quay.io/kahlai/sample-app:ubi9-native
```

Example output:
```
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2025-05-03 01:26:06,259 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT native (powered by Quarkus 3.21.3) started in 0.296s. Listening on: http://0.0.0.0:8080
2025-05-03 01:26:06,267 INFO  [io.quarkus] (main) Profile prod activated. 
2025-05-03 01:26:06,267 INFO  [io.quarkus] (main) Installed features: [cdi, rest, smallrye-context-propagation, vertx]
```


## Conclusion
We have successfully run a simple quarkus java application using UBI Micro and JRE.
Size and attack surface is reduced to UBI micro packages + JRE itself. 