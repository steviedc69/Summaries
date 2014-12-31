# Hadoop
========

## Introduction

Volume, Variety and Velocity are 3 problems that relational databases have trouble with. Hadoop tries to solve this with HDFS, in which it uses distributed data, redundancy and namenodes to guarantee uptime and fast computation

## HDFS

### Intro

1 Namenode and multiple datanodes.

Data is distributed in blocks of (default) 64MB.

Data is duplicated on the drives to reduce problems on hardware failure.

Namenode has duplicates too.

### Commands

View contents of HDFS

    hadoop fs -ls

Put file in HDFS

    hadoop fs -put file

A few bash commands work in the HDFS:

    hadoop fs -command
    tail
    cat
    cp
    mv
    rm
    mkdir
    du

To check the health and duplication status of the HDFS

    hdfs dfsadmin -report

To modify block sizes

    src/hadoop-common-project/hadoop-common/src/test/resources/core-site.xml