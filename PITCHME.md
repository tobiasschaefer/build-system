Build Automation in Action

![Build System](http://www.clker.com/cliparts/d/9/D/J/z/t/five-cogwheels.svg)

Tobias Schäfer, NovaTec Consulting GmbH


---

### Agenda

* Begriffsklärung
* Warum Continuous Integration?
* Warum Continuous Deployment?
* Das Open Source Projekt "Educama"
* Build Automatisierung mit Travis CI
* In Aktion: Pull Request erstellen und mergen am konkreten Beispiel

---

### Begriffsklärung (1/3)

> Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day.

_Quelle: https://www.thoughtworks.com/continuous-integration_

---

### Begriffsklärung (2/3)

> Continuous delivery (CD) is a software engineering approach in which teams produce software in short cycles, ensuring that the software can be reliably released at any time

_Quelle: https://en.wikipedia.org/wiki/Continuous_delivery_

---

### Begriffsklärung (3/3)

> Continuous deployment (CD) is a strategy for software releases wherein any commit that passes the automated testing phase is automatically released into the production deployment.

_Quelle: http://searchitoperations.techtarget.com/definition/continuous-deployment_

---

## Warum Continuous Integration?

## Warum Continuous Deployment? 

---

### Warum Continuous Integration?

* Abschied von langwierigen Integrationen
* Erhöhte Sichtbarkeit verbessert die Kommunikation
* Fehler werden frühzeitig erkannt
* Weniger Wartezeiten, ob der Code wirklich funktioniert
* Weniger Integrationsprobleme führen zu schnelleren Auslieferungen

_Quelle: https://www.thoughtworks.com/continuous-integration_

---

### Warum Continuous Deployment? 

* Weniger Risiko beim Deployment
* Nachweisbarer Fortschritt
* Frühe Rückmeldung von Anwendern

_Quelle: https://martinfowler.com/bliki/ContinuousDelivery.html_

---

### Das Open Source Projekt "Educama"

"Educama is a showcase demonstrating Case Management and Business Process Management based on Camunda BPM, Spring Boot, and Angular. [...] Educama was founded in 2016 in the context of a research cooperation between Reutlingen University and NovaTec Consulting GmbH."

_Quelle: http://educama.org_

---

## Continuous Integration & Deployment mit Travis CI

* CI Konfiguration von [Travis CI](https://travis-ci.org/) mittels .travis.yml
* CD Konfiguration von [Cloud Foundry](https://docs.cloudfoundry.org/) mittels manifest.yml
* Alternativen zu Travis CI? [Jenkins](https://jenkins.io/), [Bamboo](https://www.atlassian.com/software/bamboo), ...

+++

```bash
# Configuration file for Travis CI, see https://docs.travis-ci.com
language: java
jdk:
- oraclejdk8
env:
  global:
  - NODE_VERSION: 6
before_script:
  - export DISPLAY=:99.0 && sh -e /etc/init.d/xvfb start #start xvfb so that we can start Firefox, see https://docs.travis-ci.com/user/gui-and-headless-browsers/#Using-xvfb-to-Run-Tests-That-Require-a-GUI
before_install:
- nvm install $NODE_VERSION
script:
- (cd backend && ./build.sh)
  && (cd frontend && ./build.sh)
  && (cd acceptance-tests && ./build.sh && ./upload-serenity-reports.sh)
addons:
  #Browser is used by the acceptance tests
  firefox: '42.0'
cache:
  timeout: 604800 #1 week
  directories:
  - "$HOME/.m2"
deploy:
  provider: cloudfoundry
  username: educama-core@novatec-gmbh.de
  password:
    secure: T3gblw+Bikg+4ELFsqHxSV96RngijEJqkfhsjkrprU4xff0YGiKgGWTcWmemAYxdlKogL3u6FT0TcFP95hdYDXUWggtzZKZGq6LLALJHRjITDaIXfq1xvDhaC8Q54t28UKM+mH8aO7XxnUOjrlHSScJNvVFDtJKorvf7KJsz/0rzEc7km7b0Yrnonj2QlPwycBPMBYTgo5XDuPMMskJ/fJG4Fi/MYLYlEUeyb/QCJawmKcbrjItDoMxVtmZnsHKE5Dyej+aGQj906dPCUTMN1cS4bi/vpQBhtGgo9FqD5QQm3fvYlmmZsjvMUHh1jLO8peBvuRpcUkBuZ/naLYy9mXmy1KfgbyPPczG1wNq3sl7mgvxdbypQuhiedgJGY6j66FEL9BQPJdPdoQ4u2faBwCY27mQBcI0DfEwfqzpYyv2Ke7zmv5O6eobSNj9i/W7+EhackWqEZEf8Hxh89/UrGYw00qLP6mUPI3IRub6Ds6kQYHOcmV4KPK3iRNb9+mFQYgftHzyHIuZUmskbE5ccsMXRm813ommeOq25XH/DbO1LvOavf3KQUoEc3+k6fLlp4B3OU07+dlhPhPXa76nGESofgXKtswGrXx17ypKfNNBpBqsenn7VeZUU2blgCn/bfAreRsCKvCxx5FmFoyt/+oeamHrw5oh/6Uz+qdO3yHk=
  api: https://api.ng.bluemix.net
  organization: '"NovaTec Consulting GmbH"'
  space: Educama-Development
  on:
    repo: Educama/Showcase
```

+++

```bash
# Configuration file for Cloud Foundry, see https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html
applications:
- name: educama-backend
  path: backend/target/educama-backend-exec.jar
  buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.10
  memory: 768M
  disk_quota: 256M
  env:
    SPRING_H2_CONSOLE_ENABLED: false
    FLYWAY_LOCATIONS: classpath:db/migration/common,classpath:db/migration/mysql
  services:
   - educama-mysql-db
- name: educama
  path: frontend/dist/educama-frontend.zip
  buildpack: https://github.com/cloudfoundry/staticfile-buildpack.git#v1.3.16
  memory: 64M
  disk_quota: 64M
```
---

### In Aktion: Pull Request erstellen und mergen am konkreten Beispiel

* Repository klonen
* Änderung machen
* Änderung lokal commiten
* Änderung auf Fork pushen (Travis CI läuft los)
* Pull Request erstellen (Travis CI läuft los)
* Pull Request mergen (Travis CI läuft los)

---

### Tipps für die Praxis in Studentischen Projekten

* Stufe 1: Quellcode von studentischen Projekten auf Github hosten (Kostenlos für Open Source Projekte)
* Stufe 2: Travis CI einrichten (Kostenlos für Open Source Projekte)
* Stufe 3: Deployment in der Cloud ([IBM Bluemix](https://console.ng.bluemix.net/registration/) oder [Pivotal Web Services](https://run.pivotal.io/))

---

### Danke

https://gitpitch.com/tobiasschaefer/educama-build-automation

+++

### Themensammlung

* Neues Projekt mit Spring Initializr erstellen und mit Travis CI verknüpfen
* Git Magic (Amend, Fixup, Reorder, Force Push, ...)
* Services binden

