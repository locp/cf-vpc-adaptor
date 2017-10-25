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
* Public zone A subnet (192.168.0.0/24) called _${AWS::StackName}_*-publicA*
* Public zone B subnet (192.168.1.0/24) called _${AWS::StackName}_*-publicB*
* Private zone A subnet (192.168.0.2/24) called _${AWS::StackName}_*-privateA*
* Private zone B subnet (192.168.0.3/24) called _${AWS::StackName}_*-privateB*
* An external security group that allows HTTP and HTTPS to the public subnet.
  Called _${AWS::StackName}_*-external*.
* An internal security group that allows TCP/8080 from the public subnet to
  the private subnet. Called _${AWS::StackName}_*-internal*.

An alternative method that will break out this template into two reusable
templates will be presented.

## Aims

Provide a basic example of a stack that has been converted into
more generic and reusable stacks along with enough documentation to get
readers of this document started on writing more advanced infrastructure based
on the same principles.  This will provide a starting point to assist in
following AWS suggested best practices for CloudFormation, namely
"[Use Cross-Stack References to Export Shared Resources](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html#cross-stack)"

## Method

Firstly split the resources that are normally in the VPC tab of the AWS console
into a stack called **vpc** for creating a new VPC and **vpc-adaptor** for
referring to an existing VPC.  This will include both the VPC and the public and
private subnets.  The identity of the resources created in this stack will be
[exported](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html)
for consumption by the other stack.

The advantage of this approach is that resources created in the subsequent
stack or stacks (to be described shortly) can be run from templates unaltered
and refer to either VPC resources that have recently been created or were
created by a method other than CloudFormation.

The subsequent stack will be called **ec2** and will contain the security
groups.  These will integrate with resources in the **vpc/vpc-adaptor** stack
by referring to the resource identities via the
[`Fn::ImportValue`](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html)
function.  There will also be documentation and examples included in the
[Results](#results) section.

## Results

The [`monolithic-stack.yaml`](monolithic-stack.yaml) file was split into
multiple files called [`vpc.yaml`](vpc.yaml) (for creating a new VPC)
[`vpc-adaptor.yaml`](vpc-adaptor.yaml) (for working with an existing VPC)
and [`ec2.yaml`](ec2.yaml) which is a small example of importing values from
either of the VPC stacks.

For a VPC stack called `example`, the following values would be exported.

|Exported Resource Name  |Description               |Example Value                  |
|------------------------|--------------------------|-------------------------------|
|example-vpc             |The ID of the VPC         |vpc-7cd58215                   |
|example-private-subnet-a|ID of Zone A Edge Subnet  |subnet-0de0a176                |
|example-private-subnet-b|ID of Zone B Edge Subnet  |subnet-ba3eacf7                |
|example-private-subnets |CSV of Edge Subnets       |subnet-0de0a176,subnet-ba3eacf7|
|example-public-subnet-a |ID of Zone A Inside Subnet|subnet-86e3a2fd                |
|example-public-subnet-b |ID of Zone A Inside Subnet|subnet-7722b03a                |
|example-public-subnets  |CSV of Inside Subnets     |subnet-86e3a2fd,subnet-7722b03a|

Therefore you could use `vpc.yaml` to create a stack called **example** and then
use `vpc-adaptor.yaml` to create a stack called **example2** that refers to
the resources created in the **example** stack.  The outputs of the two stacks
should be identical.  Subsequently, creating two stacks using the `ec2.yaml`
template and referring each of them to **example** and **example2** would also
result in identical resources being created within the stacks.

## Limitations

Currently this example only covers the creation of a public subnet and
a private subnet within each of two AZs.  Support for additional AZs
will be added in future.

There also is not functional difference between the public (edge) subnet and
the private (inside) subnet.  Again this will be expanded upon in future.

During the development process, it was attempted to create a single template for
the VPC that would allow the user to select as to whether a new VPC should be
created or use existing resources.  One side effect of this was that existing
resources needed to be selected (even though they would be logically ignored)
when creating a new VPC.  This would of course be impossible if the user was
deploying into an account/region that currently had no previous VPCs
configured.

In the original monolithic stack, the CIDR address was
gathered using the `Fn::GetAtt` function.  This function
[can not call another function](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html#w2ab2c21c28c27c13)
for the logical resource name.  Therefore we have had to hardcode in the
values for [RFC 1918](https://en.wikipedia.org/wiki/Private_network) subnets.
