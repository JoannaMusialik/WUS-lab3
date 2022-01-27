# WUS-lab3

## Członkowie zespołu
- Antonina Lewicka
- Joanna Musialik
- Paweł Gosk
- Robert Żurawski

## Wymagania
Instalacja poniższych pakietów:

- gcloud
- kubectl
- helm

Przed rozpoczęciem (po pierwszej instalacji) należy również dokonać pierwotnej konfiguracji gcloud-a za pomocą komendy:
- gcloud init


## Punkt pierwszy

### Ustawienie zmiennych

Można tego dokonać poprzez odpowiednią modyfikację pliku env.sh oraz wykonanie komendy:

- source env.sh

W pliku należy dostosować do swoich potrzeb następujące zmienne:
- CLOUDSK_CORE_PROJECT - id swojego projektu 
- CLOUDSDK_COMPUTE_REGION - region dla swojego projektu
- CLOUDSDK_COMPUTE_ZONE - strefa dla swojego projektu
- CLUSTER_NAME - nazwa klastra

### Dostosowanie repozytorium spring-petclinic-cloud

Przed wdrożeniem konieczna była modyfikacja repozytorium spring-petclinic. By zmienić ilość pode'ów dla serwisu, zmodyfikowano ilość replik w plikach .yaml znajdujących się w folderze spring-petclinic-cloud/k8s. 

W celu lepszej wydajności można również zmodyfikować parametr pamięci i CPU w tych plikach zgodnie z potrzebami. 


### Wdrożenie aplikacji spring-petclinic w wersji mikroserwisowej
- uruchomienie skryptu scripts/create.sh
- ustawienie zmiennej środowiskowej: export REPOSITORY_PREFIX=<DOCKERHUB_USER>
- cd spring-petclinic-cloud
- inicjalizacja przestrzeni nazw: kubectl apply -f k8s/init-namespace
- inicjalizacja serwisów: kubectl apply -f k8s/init-services
- instalacja baz danych
		helm repo add bitnami https://charts.bitnami.com/bitnami
		helm repo update
		helm install vets-db-mysql bitnami/mysql --namespace spring-petclinic --version 8.8.8 --set auth.database=service_instance_db
		helm install visits-db-mysql bitnami/mysql --namespace spring-petclinic  --version 8.8.8 --set auth.database=service_instance_db
		helm install customers-db-mysql bitnami/mysql --namespace spring-petclinic  --version 8.8.8 --set auth.database=service_instance_db
- ręczne wdrożenie aplikacji poprzez wykonanie skryptu spring-petclinic-cloud/deployToKubernetes.sh
- adres IP można uzyskać poprzez wykonanie komendy kubectl get svc -n spring-petclinic api-gateway

## Punkt drugi

### Plik spring-petclinic-cloud/pom.xml
Dodanie fragmentu: 

		<dependency>
        		<groupId>com.google.cloud</groupId>
        		<artifactId>google-cloud-logging-logback</artifactId>
        		<version>0.122.7-alpha</version>
		</dependency>

### Plik src/main/resources/logback-spring.xml
Dodanie fragmentu dla każdego serwisu: 

		<appender name="CLOUD" class="com.google.cloud.logging.logback.LoggingAppender">
    		<!-- Optional : filter logs at or above a level -->
    		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      		<level>INFO</level>
    		</filter>
    		<log>customers-service.log</log> <!-- Optional : default java.log -->
    		<enhancer>com.example.logging.logback.enhancers.ExampleEnhancer</enhancer> <!-- Optional -->
    		<flushLevel>WARN</flushLevel> <!-- Optional : default ERROR -->
  		</appender>

  		<root level="info">
    		<appender-ref ref="CLOUD" />
  		</root>


### Plik src/test/resources/logback-test.xml
Dodanie pliku src/test/resources/logback-test.xml z poniższą zawartością dla każdego serwisu:

		<?xml version="1.0" encoding="UTF-8"?>
		<configuration />


## Punkt trzeci

### Przygotowanie
Przed wykonaniem tego kroku należy wykonać wszystkie kroku z punktu pierwszego, z wyjątkiem ręcznego wdrożenia aplikacji za pomocą skryptu spring-petclinic-rest/deployToKubernetes.sh

#### Ustawienie sekretów
Konieczne jest ustawienie poniższych sekretów: 
- DOCKERHUB_USER - nazwa użytkownika DockerHub
- DOCKERHUB_PASSWORD - hasło użytkownika DockerHub
- GKE_SERVICE_ACCOUNT_KEY - klucz do konta serwisowego GKE z odpowiednimi uprawnieniami. 


### Konfiguracja pipeline'a
Plik konfiguracyjny dla pipeline'a to .github/workflows/commit_deployment.yml. Pipeline skonfigurowany jest w ten sposób, by był przystosowany do wdrażania aplikacji przy każdym commicie. 

