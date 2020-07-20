# cocoapods-checksum-demo
A demo project to reproduce an issue caused by checksum

TL;DR, current checksum is based on the spec file, which may potentially cause wrong version of pod being used when using branch as source.

## Background

First of all, let me describe our development flow of private pods.

During development, we will check out the source code of the private pod into a local directory, checkout a new feature branch, e.g. `feature-name`, and point the pod to a local path using `pod "pod_name", :path => "local_path"`.

After local development, we will submit beta builds (of the main App containing the private pod) for QA to test. In this stage, we will use `pod "pod_name", :git => "source_repo", :branch => "feature-name"` to reference the latest code of the feature branch. Beta builds are build and deployed on a stateful CI server.

Only when it's stable for submission, we will then bump the version, make a tag, and push it to our private specs repo.

## Steps to Reproduce The Problem

