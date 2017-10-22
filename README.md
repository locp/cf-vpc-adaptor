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

Provide a basic example of of a stack that has been converted into more
more generic and reusable stacks along with enough documentation to get
readers of this document started on writing more advanced infrastructure based
on the same principles.

## Method

Firstly split the resources that are normally in the VPC tab of the AWS console
into a stack called **vpc**.  This will include the VPC and the public and
private subnets.  The identity of the resources created in this stack will be
[exported](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html)
for consumption by the other stack.

This first stack will also have a parameter asking if the resources should be
created from scratch or if the output exports should all be the identities of
resources within an already existing VPC.  The advantage of this approach is
that resources created in the subsequent stack or stacks (to be described
shortly) can be run from templates unaltered and refer to either VPC resources
that have recently been created or were created by a method other than
CloudFormation that are expected to be added to.

The subsequent stack will be called **ec2** and will contain the security
groups.  These will refer to resources in the **vpc** stack by referring to
the resource identities via the
[`Fn::ImportValue`](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html)
function.  There will also be documentation and examples included in the
[Results](#results) section.

## Results

The [`monolithic-stack.yaml`](monolithic-stack.yaml) file was split into two
files called [`vpc.yaml`] and [`ec2.yaml`].

## Limitations

Currently this example only covers the creation of two a public subnet and
a private subnet within two AZs.  Support for additional AZs
will be added in future.

There also is not functional difference between the public (edge) subnet and
the private (inside) subnet.  Again this will be expanded upon in future.
