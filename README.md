# cocoapods-checksum-demo
A demo project to reproduce an issue caused by checksum

TL;DR, [current checksum is based on the spec file only](https://github.com/CocoaPods/Core/blob/59eaa9b1b22070d6c5ffcfa19f491c1f47594459/lib/cocoapods-core/specification.rb#L668), which may potentially cause wrong version of pod being used when using branch as source.

## Background

First of all, let me describe our development flow of private pods.

During development, we will check out the source code of the private pod into a local directory, checkout a new feature branch, e.g. `feature-name`, and point the pod to a local path using `pod "pod_name", :path => "local_path"`.

After local development, we will submit beta builds (of the main App containing the private pod) for QA to test. In this stage, we will use `pod "pod_name", :git => "source_repo", :branch => "feature-name"` to reference the latest code of the feature branch. Beta builds are build and deployed on a stateful CI server.

Only when it's stable for submission, we will then bump the version, make a tag, and push it to our private specs repo.

## Steps to Reproduce The Problem

I created [this minimal project](https://github.com/allenhsu/cocoapods-checksum-demo) to reproduce the issue.

1. Clone the repo, check out to `branch-based-pod`, run `pod install`, which will install `AFNetworking` from it's master branch.
2. Check out to `master` branch, run `pod install`, which is supposed to install the `AFNetworking` v4.0.1 as locked in Podfile.lock from the official trunk. However, because the checksum of `AFNetworking` didn't change, so pod skipped reinstalling it.

If you do it backward, there's no such issue, because branch based pod is pre-downloaded, which will hit [the check here](https://github.com/CocoaPods/CocoaPods/blob/7033bd7bb07a37041bc6e367b45660c9ac4784b8/lib/cocoapods/installer/analyzer/sandbox_analyzer.rb#L142).

This is causing some corner issues for us because we are using a local stateful CI server, Pods directory is not cleaned for each build. When switching to branches based on older release branches, the wrong cached pod may be used.

## How to Fix It

The quickest workaround for us is remove Pods directory before each build to make sure pods are installed from source, but this will make the build time longer than expected. For deployment tasks, we do remove Pods directory to make sure everything is clean. But for beta builds, we still want to keep it to accelerate the builds.

The better solution would be change how checksum is calculated, or how sandbox manifest and dependencies are compared. The checksum part is in this Core repo, while the sandbox analyzer is actually in the CocoaPods repo.

Not sure if anyone else has encountered similar problems, but I want to bring it up to see if there could be any improvements.
