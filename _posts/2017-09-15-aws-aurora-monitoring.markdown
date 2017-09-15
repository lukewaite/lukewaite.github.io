---
layout: post
title:  "AWS Aurora & CloudWatch Alarms"
date:   2017-08-21 22:00:00
categories: aws cloudwatch alarm monitoring aurora
---

Today I learned an interesting lesson. If you're using AWS Aurora, don't set your alarms (CPU, memory, etc) on
an individual instance. The reason? Aurora instances, by their very nature, are somewhat transient:
* If you have an issue with a given primary, Aurora may promote another instance to your `WRITER` role.
* If you need to perform an in-place update, you can sometimes do so by creating a new set of instances, and promoting those manually.

In either case, you'll either lose your alarms completely, if you forget to re-create them, or your alarms
for your primary instance will no longer be on your primary.

It turns out that the metrics Aurora publishes also include `DBClusterIdentifier` and `Role` dimensions.

If you choose that dimension group, then you can view metrics and set alarms based on the `WRITER` or `READER` roles,
for whatever cluster you desire. This is much more fool-proof than setting alarms per-instance.

{% include image.html url="/images/2017-09-15-aurora-cloudwatch/select-cluster-role.png" description="Metrics by DBClusterIdentifier and Role" %}
