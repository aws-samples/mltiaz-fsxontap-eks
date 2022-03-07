# Quickly create an FSx for NetApp ONTAP file system

## What is FSx for NetApp ONTAP (FSxO)
Amazon FSx for NetApp ONTAP is a fully managed service that provides highly reliable, scalable, performant, and feature-rich file storage built on NetApp's popular ONTAP file system. It provides the familiar features, performance, capabilities, and APIs of NetApp file systems with the agility, scalability, and simplicity of a fully managed AWS service.

## About the Cloudformation templates
There are two cloudformation templates provided in this repo:

**vpc-subnets.yaml**<br/>This creates a VPC with two public subnets and two private subnets, where the FSxONTAP file system will run on.

**FSxONTAP.yaml**<br/>This creates a FSxONTAP file system, with HA spanning across two private subnets.It allows you to put parameters such as StorageCapacity, ThroughputCapacity, FsxAdminPassword etc., to customized your file system.

## Architecture Diagram

![Diagram](/Architecture.png)
