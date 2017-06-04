# Jenkins Pipeline Generator

This repository contains the Jenkins file that is used on Jenkins Generator workflow and it is responsible for:
 * Generate GitHub repositories
 * Apply Giter8 Templates
 * Create ECS Docker Registry on AWS

## Setup

This project requires the following parameters from Jenkins:

- ProjectName [String]: The name of the project to create
- ProjectTemplate [Choice]: The kind of project to create, options are ()

It requires the following environment variables defined on Jenkins

JENKINS_GITHUB_CREDENTIALS_ID