# The STAR method
[<img src="../images/back.png">](../README.md)

## Introduction
- You can use the STAR method to structure the examples you give to questions, especially in interviews.
- Structure your working history on CV or LinkedIn
- and so on...

### What STAR stands for
- **Situation** - the situation you had to deal with
- **Task** - the task you were given to do
- **Action** - the action you took
- **Result** - what happened as a result of your action and what you learned from the experience

### How to use STAR (worked out one of my recent assignments)
- **Situation** - Project wanted to have a fully automated End-to-End regression test for all services participating a SAP IFRS calculation run
- **Task** - Design and Implement an Azure DevOps nightly-run pipeline to rebuild all services from scratch from the latest state of development **(GIT master branch)** for that working day, spin up an Azure Test Cluster from scratch using Terraform, deploy all services on that new cluster and trigger a full End-to-End report and report the test outcome to an MS Teams Channel. This will enable the team next morning to check the over health state of all development efforts from last working day.
- **Action** - I presented my design proposals to the team and worked out the following solution...
  1. Build, Docker package and publish Docker images for all MicroServices involved from the latest state **(GIT master branch)** of development for that particular working day.
  2. Spin up an Azure AKS cluster from scratch using Terraform IaC manifests.
  3. HELM deploy each individual MicroService to that new cluster by pulling and installing Docker images produced from 1.
  4. Trigger the End-to-End processing from a dedicated testing dataset of contracts and cashflows...
  5. Consolidate test results and send the outcome to a predefined MS Teams Channel for the team to get observed begin of day.
- **Result** - This resulted in the following overall organizational performance improvements...
  - The overall nightly tests provided a quality gate preventing bugs slipping through into UAT and production releases.
  - Bugs could get intercepted and anticipated on in an early stage immediately next day early morning when team members opened up the dedicated Team Channels.
  - Leveraged a higher level of awareness with developers to put more emphasis on individual testing before merging changes to master branch.
  - Higher visibility on quality and stability of last day pull requests.

## References
- [What STAR stands for](https://nationalcareers.service.gov.uk/careers-advice/interview-advice/the-star-method)
- [How to write a STARish CV](https://nationalcareers.service.gov.uk/careers-advice/cv-sections)

[<img src="../images/back.png">](../README.md)