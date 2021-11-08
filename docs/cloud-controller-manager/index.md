# Cloud Controller Manager

This document describes the necessary changes that must be made to add a new
[Cloud Controller Manager (CCM)](https://kubernetes.io/docs/concepts/architecture/cloud-controller/)
to OpenShift. It does not cover the details of writing a CCM, for information
about implementing the controller please see the official Kubernetes documentation
on [Developing Cloud Controller Manager](https://kubernetes.io/docs/tasks/administer-cluster/developing-cloud-controller-manager/)
as a starting point.

## Add the CCM repository to OpenShift

One of the first things to do is copy the source code into the OpenShift organization
on GitHub. This process is described in more detail in the
[Creating a GitHub Repository in the OpenShift Organization](../procedures/creating-an-openshift-repository)
document.

Having the source code in the OpenShift organization will allow Red Hat's continuous
integration tooling access to it for running tests, building images, and including in
the official releases.

## Configure basic repository integrations

After the repository is copied to the OpenShift organization, there are a few basic
continuous integration tasks that should be configured. The most important of these
initially are the job to permit automated merging through
[Tide](https://github.com/kubernetes/test-infra/tree/master/prow/tide)
and the job to create container images on commits.

OpenShift uses [Prow](https://github.com/kubernetes/test-infra/tree/master/prow) for
continuous integration testing much like the Kubernetes community does. The configurations
are contained in the [openshift/release](https://github.com/openshift/release)
repository. This is where pull requests will need to be made to enable integrations
for the new CCM.

For a deeper understanding of the OpenShift continuous integration tooling, the
[OpenShift CI Docs](https://docs.ci.openshift.org/) are the authoritative source.
In specific for this initial task, the
[Bootstrapping Configuration for a new Repository](https://docs.ci.openshift.org/docs/how-tos/onboarding-a-new-component/#bootstrapping-configuration-for-a-new-repository) should be used as a guide.

As an example, here is the pull request to add the CCM for Alibaba Cloud to the
OpenShift Prow. It configures the Tide mechanics as well as a pre-submit job to
check the Go language formatting and a post-submit job to build the container images.

* [openshift/release #19947](https://github.com/openshift/release/pull/19947)

## Integrating with other OpenShift components

In OpenShift there are several operators which work together to ensure that the CCM
is properly configured and also that new nodes which join the cluster are similarly
configured for the infrastructure provider. To ensure that the new CCM and provider
are properly recognized by all components there are several repositories which must
be modified.

### openshift/api

Before progressing with the OpenShift integrations, the new infrastructure provider
will need to be present in the OpenShift API as part of the `infrastructure.config.openshift.io`
definition.

_**TODO: add link to installer doc step that describes updating openshift/api**_

### openshift/library-go

In order for the new infrastructure provider to be recognized by the Cluster
Cloud Controller Manager Operator, and other operators, the
[OpenShift library-go project](https://github.com/openshift/library-go) must be updated
so that the `IsCloudProviderExternal` function returns the proper response.

As an example, here is the pull request to update the `IsCloudProviderExternal` function
to add support for IBM Cloud.

* [openshift/library-go #1161](https://github.com/openshift/library-go/pull/1161)

It is important to note that any changes to library-go must be vendored into the projects
which are dependent upon library-go. This process can often involve several repositories
that must be updated before all the components will work together. To make tracking
these changes easier, an issue can be used to monitor the individual changes. For example,
this is an issue which tracked changes for the GCP and vSphere updates:

* [openshift/cluster-cloud-controller-manager-operator #135](https://github.com/openshift/cluster-cloud-controller-manager-operator/issues/135)

### openshift/cluster-cloud-controller-manager-operator

OpenShift uses the [Cluster Cloud Controller Manager Operator](https://github.com/openshift/cluster-cloud-controller-manager-operator)(CCCMO)
to manage the deployment and maintenance of the CCMs. This operator will deploy
the individual containers of the CCM and ensure their health and continued
operation during the cluster lifecycle.

Adding a new infrastructure provider will require creating a package within the
CCCMO that will contain the code for deploying the new components as well as the
templated manifests for them. For a deeper discussion of integrating with the
CCCMO, please see the
[developer documenation](https://github.com/openshift/cluster-cloud-controller-manager-operator/blob/master/docs/dev/cloud-provider-integration.md)
from that repository.

As an example, here are two pull requests that added the IBM Cloud CCM to the
CCCMO. The first pull request adds the primary code changes and manifests for deployment.
The second pull request adds the CCM image references for the CCCMO to utilize
when deploying from an OpenShift release payload.

* [openshift/cluster-cloud-controller-manager-operator #97](https://github.com/openshift/cluster-cloud-controller-manager-operator/pull/97)
* [openshift/cluster-cloud-controller-manager-operator #105](https://github.com/openshift/cluster-cloud-controller-manager-operator/pull/105)

