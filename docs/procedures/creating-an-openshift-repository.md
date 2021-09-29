# Creating a GitHub Repository in the OpenShift Organization

A common task when adding a new component to OpenShift is creating a GitHub
repository in the [OpenShift organization](https://github.com/openshift). This
is where all the code that becomes part of an OpenShift release lives. To
accomplish creating a new repository, a Red Hat representative will need to make a
Jira request on the internal `DPP Board`.

For infrastructure implementors:
1. Contact your Red Hat representative about creating a new repository.
1. Have the name, description, and license information for the new repository
    ready to share.
1. If you would like to have the repository forked from an existing repository
    let your contact know this at the beginning of the process.

For Red Hat representatives:
1. Create a new Jira Request on the `DPP Board`, the following is an example
    request for the GCP CCM:
    ```
    Repository name: cloud-provider-gcp
    Description: Kubernetes Cloud Controller Manager for Google Cloud Platform
    Programming Language (for .gitignore - optional): go
    License: Apache License 2.0
    Public repo: Public
    Who needs write access: OpenShift Team Cloud(https://github.com/orgs/openshift/teams/openshift-team-cloud)
    Read access: Everybody
    Additional info: Please fork https://github.com/kubernetes/cloud-provider-gcp there.
    ```
