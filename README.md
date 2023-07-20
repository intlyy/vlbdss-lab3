# lab3-cloud-service-availability
In this Lab, we will deploy a complete set of monitoring and alert services, and use a series of methods to improve the availability of services

## Introduction

> What is availability?

> How to Measure Availability?

> What are the common factors that affect the availability of services, and how to solve these problems

> What are the differences in ensuring service availability between cloud and non-cloud.

## Learning Objectives

- Learn the definition and measurement of availability
- Learn some best practices about high availability

## Pre-requisites
Lab3 has the same prerequisites as lab1. it will speed up the progress of lab3 if you have completed lab1.

- An AWS account
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/): the standard Kubernetes command line interface
- Node.js 16. Use nvm to [install Node.js 16](https://github.com/nvm-sh/nvm#installing-and-updating):

  ```bash
  $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
  $ nvm install 16
  ```

- Basic knowledge of AWS, Kubernetes, RDBMS
- VPN for connecting to AWS API and GitHub

## Syllabus

1. Create a new GitHub repository base on [this template](https://github.com/vldbss-2023/lab3-cloud-service-availability)
   * Clone the newly created repository
   * In the root directory of the repository, run `make install` to install the dependencies
2. (20 min) Create an EKS cluster [`1-create-an-eks-cluster`](./1-create-an-eks-cluster/README.md)
3. (30 min) Deploy observability service
   Operator [`2-deploy-observability-service`](./2-deploy-observability-service/README.md)
4. (30 min) Find out issues about availability[`3-find-out-issues-aboud-availability`](./3-find-out-issues-aboud-availability/README.md)
5. (40 min) Optimize observability service
   Operator [`4-optimize-observability-service`](./4-optimize-observability-service/README.md)
6. Bonus: For the questions in the third step, is there any better solution?

---

## AWS billing price

This lab will incur charges under the aws account, described in detail at: 

- New EKS cluster control plane, **_1_** cluster x **_0.10_** USD per hour
- Two EKS worker EC2 `t2.medium` instances, **_2_** instances * **_0.0464_** USD per hour
- 4 EBS of size 1 GiB, with negligible cost

Total **_0.1928_** USD per hour.

## Criteria for judging
The fourth step of our lab will be hidden first, and students' thinking about the problems in the third step will be the focus of judgment
1. Complete the deployment operations in 1-create-an-eks-cluster and 2-deploy-observation-service. (0-20)
2. For each question in 3-find-out-issues-about-availability, 0-10 points can be obtained if only the text description is provided, and 0-15 points can be obtained if the code is directly modified.
   * 2.1 Text answer: Use AutoScaler of EC2. Code answer:
   * 2.2 Text answer: a.deploy multiple replicas for vmstorage; b. Deploy multiple replicas to different EC2. Code answer: a. `replicaCount: 2` b. `podAntiAffinity`
   * 2.3 Text answer: Deploy different services for different tenants, or isolate data and requests through tenants in a service. Code answer: `/select/1/prometheus` or deploy multiple vmalerts
   * 2.4 Text answer: Add a caching mechanism on the vmagent side, or establish another data link. Code answer: `statefulStorage`.
3. Complete the deployment operation in 4-optimization-observation-service. (0-20)
