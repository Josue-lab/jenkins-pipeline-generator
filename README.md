# Jenkins Pipeline Generator

This repository contains the Jenkins file that is used on Jenkins Generator workflow and it is responsible for:
 * Generate GitHub repositories
 * Apply Giter8 Templates
 * Create ECS Docker Registry on AWS

## Setup

This project requires the following parameters from Jenkins:

- GITHUB_ORGANIZATION [String]: The name of the Github organization where the projects will be created
- PROJECT_NAME [String]: The name of the project to create
- PROJECT_TEMPLATE [Choice]: The kind of project to create, options are ()

It requires the following environment variables defined on Jenkins

- JENKINS_GITHUB_CREDENTIALS_ID: Jenkins Credentials Id. You must first create the
credentials (RSA key or any other method) to access your organization repositories,
then copy your Jenkins credentials Id as this parameter.

##Note
We've detected that if we try to execute this Job, the parameterized Properties are not being picked it up. This problem is sorted after the first attempt.
