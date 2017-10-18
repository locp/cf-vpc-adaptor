# AWS CloudFormation VPC Adaptor Example

## Contents

1. [Introduction](#introduction)
1. [Aims](#aims)
1. [Method](#method)
1. [Results](#results)

## Introduction

This project is to provide an example of splitting out a monolithic
[AWS CloudFormation](https://aws.amazon.com/cloudformation/) template
into reusable, modular templates.  This means that code that references VPC
resources that have been created by the stack (using the `Ref` function)
could be reusable with some adjustment to reference resources in an already
existing VPC.

The example will take a very basic monolithic stack template that creates the
following:

* VPC (192.168.0.0/16) called _${AWS::StackName}_
* Public subnet (192.168.0.0/24) called _${AWS::StackName}_*-public*
* Private subnet (192.168.0.1/24) called _${AWS::StackName}_*-private*
* An external security group that allows HTTP and HTTPS to the public subnet.
  Called _${AWS::StackName}_*-external*.
* An internal security group that allows TCP/8080 from the public subnet to
  the private subnet. Called _${AWS::StackName}_*-internal*.

## Aims

## Method

## Results
