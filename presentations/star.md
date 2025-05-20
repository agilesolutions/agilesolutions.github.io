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

### Story no 1.
- **Situation** - SAP IFRS project was lacking fully automated regression testing for all services participating on Quarterly runs. At the moment too many bugs slipped through to higher order environments like UAT, and Production.
- **Task** - The project team decided to implement a fully automated End-to-End automated regression and QA testing pipeline, scheduled each night to test all the development efforts committed to the master branch that same day.   
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
  - Higher visibility on quality and stability of last day features delivered by the development team.

### Story no 2.
- **Situation** -
- **Task** -
- **Action** -
- **Result** -

### Story no 3.
- **Situation** -
- **Task** -
- **Action** -
- **Result** -
  
## References
- [What STAR stands for](https://nationalcareers.service.gov.uk/careers-advice/interview-advice/the-star-method)
- [How to write a STARish CV](https://nationalcareers.service.gov.uk/careers-advice/cv-sections)

[<img src="../images/back.png">](../README.md)