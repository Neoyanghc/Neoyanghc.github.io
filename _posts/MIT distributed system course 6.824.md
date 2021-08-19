---
layout:     post
title:      "MIT 分布式系统课程笔记 distributed system "
subtitle:   " \"世界很大，我很小\""
date:       2020-10-30 23:00:00
author:     "neo"
header-img: "img/post-bg-un.jpg"
catalog: true
tags:
    - 分布式
---

> MIT distributed system course 6.824

## Lecture 1 Introduction

#### 1.0 why we need distributed system

+ parallelism
+ more powerful performance
+ fault tolerance
+ etc....

#### 1.1 usage

+ infrastrure
+ storage
+ communication

## Lecture 2 RPC and mutiple thread

#### 1.0 Go

+ have Go is more convenience to create and collect memory than C++ 

#### 1.1 thread

+ a useful tool to control parallelism

+ when we have to write one data，we can set a LOCK
+ be warning about share data

## Lecture 3 GFS

#### 1.0 why we need google file system

+ big storage
+ faults tolerance
+ replication 

#### 1.1 Master

+ have one data table
  + file name -> array of chunk id
  + Id  -> list of chunkservers

+ LOG on disk
+ Checkpoint on disk
+ When we need to read 
  1. name to master
  2. master get us the id
  3. return the copy

+ when we need to write
  1. ask master where is the file - name to master
  2. do we have primary？
  3. if we can not find the 17th chunk(maybe because of electricity)， we will return none
  4. we have a lot of problems of primary

## Lecture 4 Primary-Backup Replication
