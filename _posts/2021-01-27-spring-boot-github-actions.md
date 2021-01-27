---
layout: post
title: SpringBoot application with Github Actions
excerpt: >-
  Create a SpringBoot application and publish it
  as a docker container to docker hub using Github actions
date: '2020-01-27'
categories: [ SpringBoot, tutorial, Docker, Github Actions ]
tags: [SpringBoot, Kotlin, Github Actions]
thumb_img_path: images/spring-boot.png
comment_issue_id: 2
---

## SpringBoot application with Github Actions

Github actions beta have been open for anyone that would like to testing out, GitHub Actions makes it easy to automate all your software workflows,
it enables you to create CI/CD pipelines to Build, test, and deploy your code right from GitHub, having this public beta opened I could not pass the opportunity of testing it out.
I will create a guessing game application using SpringBoot Webflux, the application should store the game information using a guava cache,
Our Github pipeline should do the following:
1) Once a PR is open run integration test cases automatically.
2) Once a PR is merge into master also run the test cases from #1 but additionally create a Docker image and publish it to a docker hub 

# Prerequisities
- Java 8 / maven etc
- Docker
        
# Tech Stack:
- SpringBoot Webflux
- Kotlin
- [Klint](https://ktlint.github.io/)
- Github Actions
- [Jib](https://github.com/GoogleContainerTools/jib)

# Steps
### Create a Spring boot sample application
Let's create the application using [spring boot initilizr](https://start.spring.io/) 
 - Project: Maven Project
 - Language Kotlin
 - SpringBoot: 2.2.0 RC1 
 - Dependencies: webflux, devtools, actuator, data-r2dbc
you can also use this [link](https://start.spring.io/#!type=maven-project&language=kotlin&platformVersion=2.2.0.RC1&packaging=jar&jvmVersion=11&groupId=com.odfsoft&artifactId=spring-boot-guess-game&name=spring-boot-guess-game&description=Demo%20project%20for%20Spring%20Boot&packageName=com.odfsoft.spring-boot-guess-game) to have everything pre-fill.

- lets add some features to the empty spring boot repo, the api should:
{% highlight kotlin %}
fun doSomething(variable: Type)
{% endhighlight %}

{% highlight javascript %}
document.write("JavaScript is a simple language for javatpoint learners");
{% endhighlight %}

- Create a game, which generate a random number and store's it in session.

Request:
```shell
curl -X POST http://localhost:8080/api/games
```
response:
```json
{
    "id": "7e37b1ec-e40c-425a-9757-abf8a57ffb91",
    "guess": 461
}
```
- Get a game with the id returned in the previous request.

Request:
```shell
curl -X GET http://localhost:8080/api/game/7e37b1ec-e40c-425a-9757-abf8a57ffb91
```
Response:
```json
{
    "id": "7e37b1ec-e40c-425a-9757-abf8a57ffb91",
    "guess": 461
}
```
- receive guesses, the api will response with a 200 and message text either saying too low, too high, congratulations you guessed.

Request:
```shell
curl -X POST  http://localhost:8080/api/game/7e37b1ec-e40c-425a-9757-abf8a57ffb91/guess   -H 'Content-Type: application/json'  -d '{"guessNumber": 10}'
```
Response:
```json
{
    "message": "your guess is too low"
}
```
TODO: create a step by step guide to the previous setup.
## Dockerize the application using Jib
the project will use [Jib](https://github.com/GoogleContainerTools/jib) for dockerization, hence no Dockerfile, 
Use `./mvnw compile jib:dockerBuild` to build to a local Docker daemon, this command should create a docker image with the name `guess-game`.
lets run the docker container locally: ` docker container run --name guess-game -p 8080:8080 -d guess-game`
if you try the calls on the previous step they should still work: 
Request:

```shell
curl -X POST http://localhost:8080/api/games
```
response:
```json
{
    "id": "7e37b1ec-e40c-425a-9757-abf8a57ffb91",
    "guess": 461
}
```
### Automate the test phase using github actions
 lets make github actions run the test cases as part of the pipeline
the idea is to run all the test cases when we push to a branch, and to build and deploy when we merge into master
 - [pull request job](https://github.com/odfsoft/spring-boot-guess-game/blob/master/.github/workflows/pullrequest.yml)
 
### Automate the Docker push to a docker hub using github actions
 - [merge job](https://github.com/odfsoft/spring-boot-guess-game/blob/master/.github/workflows/merge.yml)