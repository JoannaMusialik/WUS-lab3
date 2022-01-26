# WUS-lab3

## Punkt trzeci
## Ustawienie pipeline'a wdrożeniowego

### Skopiowanie projektu
W celu ustawienia repozytorium z pipeline'm wdrożeniowym, który zapewnia wdrożenie przy każdym wykonanym commicie, należy mieć repozytorium z projektem, w którym znajduje się projekt spring-petclinic-cloud. Można tego dokonać następująco:

git clone <your_own_repo>

git clone https://github.com/spring-petclinic/spring-petclinic-cloud.git

cd spring-petclinic-cloud 

rm .git

Te zmiany następnie powinny zostać wprowadzone do własnego repozytorium (stage, commit i następnie push).

### Konfiguracja pipeline'a
Plik konfiguracyjny dla pipeline'a to .github/workflows/commit_deployment.yml


## Wygenerowanie service account 


### Ustawienie sekretów




### Rezultaty
