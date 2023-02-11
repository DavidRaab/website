---
title: "Draft Problems With Private"
date: 2023-02-11T19:03:10+01:00
draft: true
---

Consider any class with a private field. Let's say a Person class
with a private password that you cannot read.

Consider you get a password from a user and want to check if the password
is correct. You will not be able todo unless the Person class choose to
implement such a function.

How do you serialise a Person to a database? You cannot until the Person class
implements and provides this function for you.

How do you serialize to JSON? You cannot until the Person class implements
and provides this function for you.

This can go on and on. private fields considered harmfull.
