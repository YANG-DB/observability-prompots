---
layout: post
title:  "Testing Automation for OpenSearch Releases"
authors: 
  - setiah
date: 2021-10-22 01:01:01 -0700
categories: 
  - technical-post
twittercard:
  description: "This post details the automated testing process for OpenSearch 1.1 and subsequent releases."
---

OpenSearch releases many [distributions](https://opensearch.org/downloads.html) across multiple platforms as part of a new version. These distributions are of two types - the default distribution that includes all plugins, and the min distribution without any plugins. They go through a rigorous testing process across multiple teams, before they are signed off as “release ready”. It includes unit testing, integration testing to validate the behavioral integrity, backward compatibility testing to ensure upgrade compatibility with previous versions, and stress testing to validate the performance characteristics. Once successfully tested, they are marked ready for the release.

The rigorous testing process provides good confidence in the quality of the release. However, so far it has been manual and non-standardized across plugins. Each plugin team validated their component by running tests on the distribution and provided their sign-off manually. With dozens of OpenSearch plugins released as part of default distribution, the turn around time for testing was high. Also, lack of a continuous integration and testing process lead to late bug discoveries which further added to release times. The Automated testing framework solves these problems by simplifying and standardizing the testing process across all components of a release distribution.

The way it works is once a new bundle is ready, the [build-workflow](https://github.com/opensearch-project/opensearch-build/blob/1.1.0/bundle-workflow/README.md#build-from-source) (explained in this [blog](https://opensearch.org/blog/technical-post/2021/10/building-opensearch-1-1-distributions/)), kicks off the `test-orchestrator-pipeline` with [input parameters](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/jenkins/test/orchestrator/Jenkinsfile#L7-L9) that uniquely identify the bundle in S3. The [test-orchestrator-pipeline](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/jenkins/test/orchestrator/Jenkinsfile) is a [Jenkins pipeline](https://www.jenkins.io/doc/book/pipeline/) to orchestrate the test workflow, consisting of three test suites - `integ-test` (integration testing)`, bwc-test` (backward compatibility testing), `perf-test`(performance testing), to run in parallel. Each of these test suites is a [Jenkins pipeline](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/jenkins/test/testsuite/Jenkinsfile) that executes the respective test type.

Like build-workflow, these test workflows are manifest-based workflows. [integ-test](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/src/run_integ_test.py) suite reads bundle manifest file to identify the type and components of a bundle under test. It pulls all maven and build dependencies for running integration tests on the bundle from s3 (these dependencies are built as part of build-workflow and re-used for testing). After pulling the dependencies, it runs integration tests for each component in the distribution, based on the component test config defined in the [test-manifest file](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/src/test_workflow/config/test_manifest.yml). It spins a new dedicated local cluster to test each [test config](https://github.com/opensearch-project/opensearch-build/blob/3d332e568de32ea6c26b63eeec2590c01d159e35/bundle-workflow/src/test_workflow/config/test_manifest.yml#L6-L8) and tears down the cluster after the test run completes. The test and the cluster logs are published to S3 after the test workflow completes. [bwc-test](https://github.com/opensearch-project/opensearch-build/blob/9bcb801f0124f09e5ad25d07a08f22b1c63b6c60/src/run_bwc_test.py) suite runs similar to `integ-test` suite, for backward compatibility tests. Currently, it only supports backward compatibility tests for OpenSearch and anomaly-detection plugin, but there’s [ongoing effort](https://github.com/opensearch-project/opensearch-build/issues/90) to add more plugins. `perf-test` suite runs performance testing with rally tracks on a dedicated external cluster. This piece is currently [in development](https://github.com/opensearch-project/opensearch-build/issues/126). Once all test suites complete, the notifications job sends out notifications to all subscribed channels. Figure 1 illustrates how different components of the test workflow interact with each other. Figure 2 shows a sample test report generated by the integration testing workflow on a release candidate build.

**Figure 1**: Automated test workflow explained

![Figure 1: Automated test workflow]({{ site.baseurl }}/assets/media/blog-images/2021-10-01-automated-testing-for-opensearch-releases/figure1.png){: .img-fluid }

**Figure 2**: A sample test reported generated by `integ-test` workflow. `with-security` config denotes the test run with security plugin enabled, `without-security` config denotes the test run without security plugin enabled.

![Figure 2: Test report]({{ site.baseurl }}/assets/media/blog-images/2021-10-01-automated-testing-for-opensearch-releases/figure2.png){: .img-fluid }

This testing automation helps make the release process faster by providing a quick feedback loop to surface issues sooner. It also standardizes the testing process and enforces strict quality controls across all components. The code is entirely [open source](https://github.com/opensearch-project/opensearch-build) and development work is being tracked on this [project board](https://github.com/opensearch-project/opensearch-build/projects/3). We welcome your comments and contributions to make this system better and more useful for everyone.