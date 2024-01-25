# Report

## Introduction

***2-1 What are testcontainers ?***

-> Testcontainers are Java libraries that allow running Docker containers for testing


***2-2 Document your Github Actions configurations.***

- branches: [ main ]
- name: Set up JDK 17
    uses: actions/setup-java@v3
    with:
      java-version: '17'
      distribution: 'temurin'
- name: Build and test with Maven 
  run: mvn clean install --file simple-api/pom.xml

