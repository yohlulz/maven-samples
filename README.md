## Sample multi-module maven project integration with Codacy and Travis CI

[![Build status](https://travis-ci.org/toratrading/maven-samples.svg?branch=master)](https://travis-ci.org/toratrading/maven-samples/builds) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/1d5ac34e3b8e48d8b0f2f68c80499047)](https://www.codacy.com/app/yohlulz/maven-samples?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=toratrading/maven-samples&amp;utm_campaign=Badge_Grade) [![Codacy Badge](https://api.codacy.com/project/badge/Coverage/1d5ac34e3b8e48d8b0f2f68c80499047)](https://www.codacy.com/app/yohlulz/maven-samples?utm_source=github.com&utm_medium=referral&utm_content=toratrading/maven-samples&utm_campaign=Badge_Coverage)

This repository is intented as an example of how a multi-module Maven project can be setup and integrated with:
* **[Codacy](https://www.codacy.com "Codacy")** - static analysis front-end that uses tools for a number of languages. When integrated with your GitHub repo, it analyses your master branch and any other you select in the settings and gives you a summary of possible issues with your code. Not only that; whenever a new pull request is opened, it checks whether it fixes any of those issue (good) or it adds new ones (bad). This allows you to try and set a trend towards cleanliness, or at least to avoid getting further from it.
* **[Travis CI](https://travis-ci.org/ "Travis CI")** - (Wikipedia)
> hosted, distributed continuous integration service used to build and test software projects hosted at GitHubhosted, distributed continuous integration service used to build and test software projects hosted at GitHub

Both Travis CI and Codacy are integrated with GitHub.

#### Codacy integration
* General presentation : http://www.thomasbembridge.com/hosted-services-for-code-coverage-and-quality-metrics-part-1/#codacy

After loging in with your GitHub account and giving the needed permissions, go ahead and add a project from the available projects. Codacy will analyse and review the project and then provide a dashboard with all sort of statistics.
![Codacy Project review](/images/codacy_review.png?raw=true "Coday Project review")
![Codacy Project dashboard](/images/codacy_dashboard.png?raw=true "Codacy Project dashboard")

Besides static code analysis, Codacy supports code coverage reports as well which will be provided by Travis after building and running the tests. For that a *Project API* integration is needed so that Codacy accepts coverage reports sent by Travis.
![Codacy Project API](/images/codacy_api.png?raw=true "Codacy Project API")

#### Travis CI
For the Travis CI integration, you'll need to login with GitHub account, acces permissions and select a project for integration.

The API token provided by the Project API integration from Codacy will have to be set up in Travis as an env variable.
![Travis Project API setup](/images/travis_env.png?raw=true "Travis Project API setup")

For the rest of the confguration, Travis needs a `.travis.yml` file. This file contains info about your environment and the actual build steps.
For example a Java based project configuration has:
```yaml
language: java
jdk: oraclejdk8
```
For Maven based projects the config contains the goals to be executed along with a setting to cache the local Maven repo directory.
```yaml
cache:
 directories:
 - $HOME/.m2

 script:  
 - mvn package
 ```
Codacy expects a single coverage report file, but in case of a multi module maven project normally a report file is generate per each module. The new versions of jacoco Maven plugin (0.7.7+) introduced a new goal [jacoco:report-aggregate](http://www.eclemma.org/jacoco/trunk/doc/report-aggregate-mojo.html "jacoco:report-aggregate") which can aggregate multiple coverage reports into a single one. 
> Creates a structured code coverage report (HTML, XML, and CSV) from multiple projects within reactor. The report is created from all modules this project depends on. From those projects class and source files as well as JaCoCo execution data files will be collected. In addition execution data is collected from the project itself. This also allows to create coverage reports when tests are in separate projects than the code under test, for example in case of integration tests.

This plugin collects all coverage reports but only from the modules that are included as a dependency. To accomodate this requirement, a **report** module was created with dependencies of the remaining modules.


From the [Codacy Coverage report tool](https://github.com/codacy/codacy-coverage-reporter/#travis-ci "Codacy Coverage report tool"), some more steps need to be added to report code coverage as well:
```yaml
before_install:
  - sudo apt-get install jq
  - wget -O ~/codacy-coverage-reporter-assembly-latest.jar $(curl https://api.github.com/repos/codacy/codacy-coverage-reporter/releases/latest | jq -r .assets[0].browser_download_url)
after_success:
  - java -cp ~/codacy-coverage-reporter-assembly-latest.jar com.codacy.CodacyCoverageReporter -l Java -r report/target/site/jacoco-aggregate/jacoco.xml
```
- This downloads the reporter jar which is used in a post step to actually send the coverage report to Codacy

The final `.travis.yml` config file look like this:
```yaml
language: java
sudo: false
jdk:
 - oraclejdk8

before_install:
  - sudo apt-get install jq
  - wget -O ~/codacy-coverage-reporter-assembly-latest.jar $(curl https://api.github.com/repos/codacy/codacy-coverage-reporter/releases/latest | jq -r .assets[0].browser_download_url)
after_success:
  - java -cp ~/codacy-coverage-reporter-assembly-latest.jar com.codacy.CodacyCoverageReporter -l Java -r report/target/site/jacoco-aggregate/jacoco.xml
install: true
cache:
 directories:
 - $HOME/.m2 
script:  
 - mvn verify -P jacoco
```

Both Travis CI and Codacy provide status Badges that can be integrated with GitHub:
![Codacy Badge](/images/codacy_badge.png?raw=true "Codacy Badge")




















