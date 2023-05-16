# Chapter 1. Red Hat JBoss Web Server Operator

An Operator is a Kubernetes-native application that makes it easy to
manage complex stateful applications in Kubernetes and OpenShift
environments. Red Hat JBoss Web Server (JWS) provides an Operator to
manage JWS for OpenShift images. You can use the JWS Operator to create,
configure, manage, and seamlessly upgrade instances of web server
applications in OpenShift.

Operators include the following key concepts:

- The *Operator Framework* is a toolkit to manage Operators in an effective, automated, and scalable way. The Operator Framework consists of three main components:
    - You can use *OperatorHub* to discover Operators that you want to install.
    - You can use the *Operator Lifecycle Manager* (OLM) to install and manage Operators in your OpenShift cluster.
    - You can use the *Operator SDK* if you want to develop your own custom Operators. 

- An *Operator group* is an OLM resource that provides multitenant configuration to OLM-installed Operators. An Operator group selects target namespaces in which to generate role-based access control (RBAC) for all Operators that are deployed in the same namespace as the `OperatorGroup` object.

- *Custom resource definitions* (CRDs) are a Kubernetes extension mechanism that Operators use. CRDs allow the custom objects that an Operator manages to behave like native Kubernetes objects. The JWS Operator provides a set of CRD parameters that you can specify in custom resource files for web server applications that you want to deploy.

This document describes how to install the JWS Operator, deploy an
existing JWS image, and delete Operators from a cluster. This document
also provides details of the CRD parameters that the JWS Operator
provides.

**Note:**
Before you follow the instructions in this guide, you must ensure that
an OpenShift cluster is already installed and configured as a
prerequisite. For more information about installing and configuring
OpenShift clusters, see the OpenShift Container Platform
[Installing](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.9/html-single/installing/index) guide.

For a faster but less detailed guide to deploying a prepared image or
building an image from an existing image stream, see the JWS Operator
[QuickStart](https://github.com/web-servers/jws-operator/blob/2.0.x/QuickStart.md) guide.

**Important**
Red Hat supports images for JWS 5.4 or later versions only.

**Additional resources**

- [Understanding Operators](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.9/html-single/operatorsindex#understanding-operators)
- [Installing and configuring OpenShift Container Platform clusters](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.9/html-single/installing/index)

# Chapter 2. What is new in the JWS Operator 2.0 release?

The JWS Operator 2.0 release provides level-2 Operator capabilities such
as seamless integration. JWS Operator 2.0 also supports Red Hat JBoss
Web Server metering labels and includes some enhanced Custom Resource
Definition (CRD) parameters.

### Level-2 Operator capabilities

JWS Operator 2.0 provides the following level-2 Operator capability
features:

-   Enables seamless upgrades
-   Supports patch and minor version upgrades
-   Manages web servers deployed by the JWS Operator 1.1.x.

### Enabling level-2 seamless integration for new images

The `DeploymentConfig` object definition includes a trigger
that OpenShift uses to deploy new pods when a new image is pushed to the
image stream. The image stream can monitor the repository for new images
or you can instruct the image stream that a new image is available for
use.

**Procedure**

1.  In your project namespace, create an image stream by using the
    `oc import-image` command to import the tag and other
    information for an image.

    For example:

    ``` screen
    oc import-image <my-image>-imagestream:latest \
    --from=quay.io/$user/<my-image>:latest \
    --confirm
    ```

    In the preceding example, replace each occurrence of
    `<my-image>` with the name of the image that you want to
    import.

    The preceding command creates an image stream named
    `<my-image>-imagestream` by importing information for the
    `quay.io/$user/<my-image>` image. For more information
    about the format and management of image streams, see [Managing
    image
    streams](https://docs.openshift.com/container-platform/4.10/openshift_images/image-streams-manage.html).
2.  Create a custom resource of the `WebServer` kind for the
    web application that you want the JWS Operator to deploy whenever
    the image stream is updated. You can define the custom resource in
    YAML file format.

    For example:

    ``` screen
    apiVersion: web.servers.org/v1alpha1
    kind: WebServer
    metadata:
      name: <my-image>
    spec:
      # Add fields here
      applicationName: my-app
      useSessionClustering: true
      replicas: 2
      webImageStream:
        imageStreamNamespace: <project-name>
        imageStreamName: <my-image>-imagestream
    ```

3.  Trigger an update to the image stream by using the
    `oc tag` command.

    For example:

    ``` screen
    oc tag quay.io/$user/<my-image> <my-image>-imagestream:latest --scheduled
    ```

    The preceding command causes OpenShift Container Platform to update
    the specified image stream tag periodically. This period is a
    cluster-wide setting that is set to 15 minutes by default.

### Level-2 seamless integration for rebuilding existing images

The `BuildConfig` object definition includes a trigger for
image stream updates and a webhook, which is either a GitHub or Generic
webhook, that enables the rebuilding of images when the webhook is
triggered by Git or GitHub.

For more information about creating a secret for a webhook, see
[Creating a secret for a generic or GitHub
webhook](#create_webhook_secret "Chapter 6. Creating a secret for a generic or GitHub webhook").

For more information about configuring a generic or GitHub webhook in a
custom resource WebServer file, see [JWS Operator CRD
parameters](#crd_parameters "Chapter 7. JWS Operator CRD parameters").

### Support for Red Hat JBoss Web Server metering labels

JWS Operator 2.0 supports the ability to add metering labels to the Red
Hat JBoss Web Server pods that the JWS Operator creates.

Red Hat JBoss Web Server can use the following metering labels:
-   `com.company: Red_Hat`
-   `rht.prod_name: Red_Hat_Runtimes`
-   `rht.prod_ver: 2022-Q4`
-   `rht.comp: JBoss_Web_Server`
-   `rht.comp_ver: 5.7.0`
-   `rht.subcomp: Tomcat 9`
-   `rht.subcomp_t: application`

You can add labels under the `metadata` section in the custom
resource `WebServer` file for a web application that you want
to deploy. For example:

``` {.programlisting .language-xml}
---
apiVersion: web.servers.org/v1alpha1
kind: WebServer
metadata:
  name: <my-image>
  labels:
    com.company: Red_Hat
    rht.prod_name: Red_Hat_Runtimes
    rht.prod_ver: 2022-Q4
    rht.comp: JBoss_Web_Server
    rht.comp_ver: 5.7.0
    rht.subcomp: Tomcat 9
    rht.subcomp_t: application
spec:
----
```

**Note:**
If you change any label key or label value for a deployed web server,
the JWS Operator redeploys the web server application. If the deployed
web server was built from source code, the JWS Operator also rebuilds
the web server application.

### Enhanced `webImage` parameter

In the JWS Operator 2.0 release, the `webImage` parameter in
the CRD contains the following additional fields:

-   `imagePullSecret`

    The secret that the JWS Operator uses to pull images from the
    repository

    **Note:**
    The secret must contain the key `.dockerconfigjson`. The
    JWS Operator mounts and uses the secret (for example,
    `--authfile /mount_point/.dockerconfigjson`) to pull the
    images from the repository. The `Secret` object definition
    file might contain server username and password values or tokens to
    allow access to images in the image stream, the builder image, and
    images built by the JWS Operator.

-   `webApp`

    A set of parameters that describe how the JWS Operator builds the
    web server application

### Enhanced `webApp` parameter

In the JWS Operator 2.0 release, the `webApp` parameter in the
CRD contains the following additional fields:

-   `name`

    The name of the web server application

-   `sourceRepositoryURL`

    The URL where the application source files are located

-   `sourceRepositoryRef`

    The branch of the source repository that the Operator uses

-   `sourceRepositoryContextDir`

    The subdirectory where the `pom.xml` file is located and
    where the `mvn install` command must be run

-   `webAppWarImage`

    The URL of the images where the JWS Operator pushes the built image

-   `webAppWarImagePushSecret`

    The secret that the JWS Operator uses to push images to the
    repository

-   `builder`

    A set of parameters that contain all the information required to
    build the web application and create and push the image to the image
    repository

    **Note:**
    To ensure that the builder can operate successfully and run commands
    with different user IDs, the builder must have access to the
    `anyuid` security context constraint (SCC).

    To grant the builder access to the `anyuid`SCC, enter the
    following command:

    `oc adm policy add-scc-to-user anyuid -z builder`

    The `builder` parameter contains the following fields:

    -   `image`

        The image of the container where the web application is built
        (for example, `quay.io/$user/tomcat10-buildah`)

    -   `imagePullSecret`

        The secret (if specified) that the JWS Operator uses to pull the
        builder image from the repository

    -   `applicationBuildScript`

        The script that the builder image uses to build the application
        `.war` file and move it to the `/mnt`
        directory

        **Note:**
        If you do not specify a value for this parameter, the builder
        image uses a default script that uses Maven and Buildah.


**Additional resources**
-   [Operator Capability Levels](https://operatorframework.io/operator-capabilities/)

# Chapter 3. JWS Operator installation from OperatorHub

You can install the JWS Operator from OperatorHub to facilitate the
deployment and management of JBoss Web Server applications in an
OpenShift cluster. OperatorHub is a component of the Operator Framework
that you can use to discover Operators that you want to install.
OperatorHub works in conjunction with the Operator Lifecycle Manger
(OLM), which installs and manages Operators in a cluster.

You can install the JWS Operator from OperatorHub in either of the
following ways:

-   [Use the OpenShift web
    console](#operator_install_console "3.1. Installing the JWS Operator by using the web console")
-   [Use the `oc` command-line
    tool](#operator_install_tool "3.2. Installing the JWS Operator by using the command line").

## 3.1. Installing the JWS Operator by using the web console

If you want to install the JWS Operator by using a graphical user
interface, you can use the OpenShift web console to install the JWS
Operator.

**Note:**
When you install the JWS Operator by using the web console, and the
Operator is using `SingleNamespace`installation mode, the
`OperatorGroup` and `Subscription` objects are
installed automatically.

**Prerequisites**

-   You have deployed an OpenShift Container Platform cluster by using
    an account with cluster administrator and Operator installation
    permissions.

**Procedure**

1.  Open the web console and select **Operators >
    OperatorHub**.

2.  In the **Filter by keyword** search field, type
    \"JWS\".

3.  Select the JWS Operator.

4.  On the **JBoss Web Server Operator** menu, select
    the **Capability level** that you want to use and
    click **Install**.

5.  On the **Install Operator** page, perform the
    following steps:

    a.  Select the **Update channel** where the JWS
        Operator is available.

        **Note:**
        The JWS Operator is currently available through one channel
        only.

    b.  Select the **Installation mode** for the Operator.

        You can install the Operator to all namespaces or to a specific
        namespace on the cluster. If you select the specific namespace
        option, use the **Installed Namespace** field
        to specify the namespace where you want to install the Operator.

        **Note:**
        If you do not specify a namespace, the Operator is installed to
        all namespaces on the cluster by default.

    c.  Select the **Approval strategy** for the
        Operator.

        Consider the following guidelines:

        -   If you select **Automatic** updates, when
            a new version of the Operator is available, the OLM upgrades
            the running instance of your Operator automatically.
        -   If you select **Manual** updates, when a
            newer version of the Operator is available, the OLM creates
            an update request. As a cluster administrator, you must then
            manually approve the update request to ensure that the
            Operator is updated to the new version.

6.  Click **Install**.

    **Note:**
    If you have selected a **Manual** approval strategy, you must approve the install plan before the installation is complete.

    The JWS Operator then appears in the **Installed Operators** section of the **Operators** tab.

## 3.2. Installing the JWS Operator by using the command line

If you want to install the JWS Operator by using a command-line
interface, you can use the `oc` command-line tool to install
the JWS Operator.

The steps to install the JWS Operator from the command line include
verifying the supported installation modes and available channels for
the Operator and creating a Subscription object. Depending on the
installation mode that the Operator uses, you might also need to create
an Operator group in the project namespace before you create the
Subscription object.

**Prerequisites**

-   You have deployed an OpenShift Container Platform cluster by using
    an account with Operator installation permissions.
-   You have installed the `oc` tool on your local system.

**Procedure**

1.  To inspect the JWS Operator, perform the following steps:

    a.  View the list of JWS Operators that are available to the cluster
        from OperatorHub:

        ``` screen
        $ oc get packagemanifests -n openshift-marketplace | grep jws
        ```

        The preceding command displays the name, catalog, and age of
        each available Operator.

        For example:

        ``` screen
        NAME            CATALOG             AGE
        jws-operator    Red Hat Operators   16h
        ```

    b.  Inspect the JWS Operator to verify the supported installation
        modes and available channels for the Operator:

        ``` screen
        $ oc describe packagemanifests jws-operator -n openshift-marketplace
        ```

2.  Check the actual list of Operator groups:

    ``` screen
    $ oc get operatorgroups -n <project_name>
    ```

    In the preceding example, replace `<project_name>` with your OpenShift project name.

    The preceding command displays the name and age of each available
    Operator group.

    For example:

    ``` screen
    NAME       AGE
    mygroup    17h
    ```

3.  If you need to create an Operator group, perform the following
    steps:

    **Note:**
    If the Operator you want to install uses `SingleNamespace`
    installation mode and you do not already have an appropriate
    Operator group in place, you must complete this step to create an
    Operator group. You must ensure that you create only one Operator
    group in the specified namespace. If the Operator you want to install uses `AllNamespaces` installation mode or you already have an appropriate Operator group in place, you can ignore this step.

    a.  Create a YAML file for the `OperatorGroup` object.

    For example:

    ``` screen
    apiVersion: operators.coreos.com/v1
    kind: OperatorGroup
    metadata:
      name: <operatorgroup_name>
      namespace: <project_name>
    spec:
      targetNamespaces:
      - <project_name>
    ```

    In the preceding example, replace `<operatorgroup_name>` with the name of the Operator group that you want to create, and replace `<project_name>` with the name of the project where you want to install the Operator. To view the project name, you can run the `oc project -q` command.

    b.  Create the `OperatorGroup` object from the YAML file:

    ``` screen
    $ oc apply -f <filename>.yaml
    ```

    In the preceding example, replace `<filename>.yaml` with the name of the YAML file that you have created for the `OperatorGroup` object.

4.  To create a Subscription object, perform the following steps:

    a.  Create a YAML file for the `Subscription` object.

    For example:

    ``` screen
    apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
        name: jws-operator
        namespace: <project_name>
    spec:
        channel: alpha
        name: jws-operator
        source: redhat-operators
        sourceNamespace: openshift-marketplace
    ```

    In the preceding example, replace `<project_name>` with the name of the project where you want to install the Operator. To view the project name, you can run the `oc project -q` command.

    The namespace that you specify must have an `OperatorGroup` object that has the same installation mode setting as the Operator. If the Operator uses `AllNamespaces` installation mode, replace `<project_name>` with `openshift-operators`, which already provides an appropriate Operator group. If the  Operator uses `SingleNamespace` installation mode, ensure that this namespace has only one `OperatorGroup` object.

    Ensure that the `source` setting matches the `Catalog Source` value that was displayed when you verified the available channels for the Operator (for example, `redhat-operators`).

    b.  Create the `Subscription` object from the YAML file:

    ``` screen
    $ oc apply -f <filename>.yaml
    ```

    In the preceding example, replace `<filename>.yaml` with the name of the YAML file that you have created for the `Subscription` object.

**Verification**

-   To verify that the JWS Operator is installed successfully, enter the
    following command:

    ``` screen
    $ oc get csv -n <project_name>
    ```

    In the preceding example, replace `<project_name>` with
    the name of the project where you have installed the Operator.

    The preceding command displays details of the installed Operator.

# Chapter 4. Deploying an existing JWS image 

You can use the JWS Operator to facilitate the deployment of an existing image for a web server application that you want to deploy in an OpenShift cluster. In this situation, you must create a custom resource `WebServer` file for the web server application that you want to deploy. The JWS Operator uses the custom resource `WebServer` file to handle the application deployment.

**Prerequisites**

-   You have [installed the JWS Operator from
    OperatorHub](#operator_install "Chapter 3. JWS Operator installation from OperatorHub").

    To ensure that the JWS Operator is installed, enter the following
    command:

    ``` screen
    $ oc get deployment.apps/jws-operator
    ```

    The preceding command displays the name and status details of the
    Operator.

    For example:

    ``` screen
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    jws-operator    1/1     1            1           15h
    ```
    **Note:**
    If you want to view more detailed output, you can use the following
    command:

    `oc describe deployment.apps/jws-operator`

**Procedure**

1.  Prepare your image and push it to the location where you want to
    display the image (for example, `quay.io/<USERNAME>/tomcat-demo:latest`).

2.  To create a custom resource file for your web server application,
    perform the following steps:

    a.  Create a YAML file named, for example,
        `webservers_cr.yaml`.

    b.  Enter details in the following format:

    ``` screen
    apiVersion: web.servers.org/v1alpha1
    kind: WebServer
    metadata:
        name: <image name>
    spec:
        # Add fields here
        applicationName: <application name>
        replicas: 2
    webImage:
       applicationImage: <URL of the image>
    ```

    For example:

    ``` screen
    apiVersion: web.servers.org/v1alpha1
    kind: WebServer
    metadata:
        name: example-image-webserver
    spec:
        # Add fields here
        applicationName: jws-app
        replicas: 2
    webImage:
       applicationImage: quay.io/<USERNAME>/tomcat-demo:latest
    ```

3.  To deploy your web application, perform the following steps:

    a.  Go to the directory where you have created the web application.

    b.  Enter the following command:

    ``` screen
    $ oc apply -f webservers_cr.yaml
    ```

    The preceding command displays a message to confirm that the web application is deployed.

    For example:

    ``` screen
    webserver/example-image-webserver created
    ```

    When you run the preceding command, the Operator also creates a route automatically.

4.  Verify the route that the Operator has automatically created:

    ``` screen
    $ oc get routes
    ```

5.  Optional: Delete the `webserver` that you created in step 3 of this procedure:

    ``` screen
    $ oc delete webserver example-image-webserver
    ```

    **Note:**
    Alternatively, you can delete the `webserver` by deleting the YAML file. For example:

    `oc delete -f webservers_cr.yaml`

**Additional resources**

-   [Route configuration: Creating an HTTP-based
    route](https://docs.openshift.com/container-platform/4.7/networking/routes/route-configuration.html)

# Chapter 5. JWS Operator deletion from a cluster 

If you no longer need to use the JWS Operator, you can subsequently
delete the JWS Operator from a cluster.

You can delete the JWS Operator from a cluster in either of the
following ways:

-   [Use the OpenShift web
    console](#operator_delete_console "5.1. Deleting the JWS Operator by using the web console").
-   [Use the `oc` command-line
    tool](#operator_delete_tool "5.2. Deleting the JWS Operator by using the command line").

## 5.1. Deleting the JWS Operator by using the web console

If you want to delete the JWS Operator by using a graphical user
interface, you can use the OpenShift web console to delete the JWS
Operator.

**Prerequisites**

-   You have deployed an OpenShift Container Platform cluster by using
    an account with `cluster admin` permissions.

    **Note:**
    If you do not have `cluster admin` permissions, you can circumvent this requirement. For more information, see [Allowing non-cluster administrators to install Operators](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.5/html/operators/administrator-tasks#olm-creating-policy).

**Procedure**

1.  Open the web console and click [**Operators > Installed
    Operators**].

2.  Select the [**Actions**] menu and click **Uninstall Operator**.

    **Note:**
    The [**Uninstall Operator**] option automatically removes the Operator, any Operator deployments, and Pods. Deleting the Operator does not remove any custom resource definitions or custom resources for the Operator, including CRDs or CRs. If the Operator has deployed applications on the cluster, or if the Operator has configured resources outside the cluster, you must clean up these applications and resources manually.

## 5.2. Deleting the JWS Operator by using the command line 

If you want to delete the JWS Operator by using a command-line interface, you can use the `oc` command-line tool to delete the JWS Operator.

**Prerequisites**

-   You have deployed an OpenShift Container Platform cluster by using
    an account with `cluster admin` permissions.

    **Note:**
    If you do not have `cluster admin` permissions, you can circumvent this requirement. For more information, see [Allowing non-cluster administrators to install Operators](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.5/html/operators/administrator-tasks#olm-creating-policy).

-   You have installed the `oc` tool on your local system.

**Procedure**

1.  Check the current version of the subscribed Operator:

    ``` screen
    $ oc get subscription jws-operator -n <project_name> -o yaml | grep currentCSV
    ```

    In the preceding example, replace `<project_name>` with
    the namespace of the project where you installed the Operator. If
    your Operator was installed to all namespaces, replace
    `<project_name>` with `openshift-operators`.

    The preceding command displays the following output, where
    `v2.0.x` refers to the Operator version (for example,
    `v2.0.0`):

    ``` screen
    f:currentCSV: {}
    currentCSV: jws-operator.v2.0.x
    ```

2.  Delete the subscription for the Operator:

    ``` screen
    $ oc delete subscription jws-operator -n <project_name>
    ```

    In the preceding example, replace `<project_name>` with
    the namespace of the project where you installed the Operator. If
    your operator was installed to all namespaces, replace
    `<project_name>` with `openshift-operators`.

3.  Delete the CSV for the Operator in the target namespace:

    ``` screen
    $ oc delete clusterserviceversion <currentCSV> -n <project_name>
    ```

    In the preceding example, replace `<currentCSV>` with the
    `currentCSV` value that you obtained in [Step
    1](#step1 "Procedure") (for example,
    `jws-operator.v2.0.x`). Replace `<project_name>`
    with the namespace of the project where you installed the Operator.
    If your operator was installed to all namespaces, replace
    `<project_name>` with `openshift-operators`.

    The preceding command displays a message to confirm that the CSV is
    deleted.

    For example:

    ``` screen
    clusterserviceversion.operators.coreos.com "jws-operator.v2.0.x" deleted
    ```
# Chapter 6. Creating a secret for a generic or GitHub webhook 

You can create a secret that you can use with a generic or GitHub
webhook to trigger application builds in a Git repository. Depending on
the type of Git hosting platform that you use for your application code,
the JWS Operator provides a `genericWebhookSecret`parameter
and a `githubWebhookSecret` parameter that you can use to
specify the secret in the custom resource file for a web application.

**Procedure**

1.  Create an encoded secret string:

    a.  Create a file named, for example, `secret.txt`.

    b.  In the `secret.txt` file, enter the secret string in
        plain text.

    For example:

    ``` screen
    qwerty
    ```

    c.  To encode the string, enter the following command:

    ``` screen
    base64 secret.txt
    ```

    The preceding command displays the encoded string.

    For example:

    ``` screen
    cXdlcnR5Cg==
    ```

2.  Create a `secret.yaml` file that defines an object of kind `Secret`.

    For example:

    ``` screen
    kind: Secret
    apiVersion: v1
    metadata:
      name: jws-secret
    data:
      WebHookSecretKey: cXdlcnR5Cg==
    ```

    In the preceding example, `jws-secret` is the name of the
    secret and `cXdlcnR5Cg==` is the encoded secret string.

3.  To create the secret, enter the following command:

    ``` screen
    oc create -f secret.yaml
    ```

    The preceding command displays a message to confirm that the secret
    is created.

    For example:

    ``` screen
    secret/jws-secret created
    ```

**Verification**

1.  Get the URL for the webhook:

    ``` screen
    oc describe BuildConfig | grep webhooks
    ```

    The preceding command generates the webhook URL in the following
    format:

    ``` screen
    https://<host>:<port>/apis/build.openshift.io/v1/namespaces/<namespace>/buildconfigs/<name>/webhooks/<secret>/generic
    ```

2.  Create a minimal JSON file named, for example,
    `payload.json`:

    ``` screen
    {}
    ```

3.  To send a request to the webhook, enter the following
    `curl` command:

    ``` screen
    curl -H "X-GitHub-Event: push" -H "Content-Type: application/json" -k -X POST --data-binary @payload.json https://<host>:<port>/apis/build.openshift.io/v1/namespaces/<namespace>/buildconfigs/<name>/webhooks/<secret>/generic
    ```

    In the preceding example, `payload.json` is the name of
    the minimal JSON file you have created.

    Replace `<host>`, `<port>`, `<namespace>`, and `<name>` in the URL string with values that are appropriate for your environment. Replace `<secret>` with the name of the secret you have created for the webhook.

    The preceding command generates the following type of webhook
    response in JSON format:

    ``` screen
    {"kind":"Build","apiVersion":"build.openshift.io/v1","metadata":{"name":"test-2","namespace":"jfc","selfLink":"/apis/build.openshift.io/v1/namespaces/jfc/buildconfigs/test-2/instantiate","uid":"a72dd529-edc6-4e1c-898e-7c0dbbea176e","resourceVersion":"846159","creationTimestamp":"2020-10-30T12:29:30Z","labels":{"application":"test","buildconfig":"test","openshift.io/build-config.name":"test","openshift.io/build.start-policy":"Serial"},"annotations":{"openshift.io/build-config.name":"test","openshift.io/build.number":"2"},"ownerReferences":[{"apiVersion":"build.openshift.io/v1","kind":"BuildConfig","name":"test","uid":"1f78fa3f-2f3b-421b-9f49-192184cc2280","controller":true}],"managedFields":[{"manager":"openshift-apiserver","operation":"Update","apiVersion":"build.openshift.io/v1","time":"2020-10-30T12:29:30Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:openshift.io/build-config.name":{},"f:openshift.io/build.number":{}},"f:labels":{".":{},"f:application":{},"f:buildconfig":{},"f:openshift.io/build-config.name":{},"f:openshift.io/build.start-policy":{}},"f:ownerReferences":{".":{},"k:{\"uid\":\"1f78fa3f-2f3b-421b-9f49-192184cc2280\"}":{".":{},"f:apiVersion":{},"f:controller":{},"f:kind":{},"f:name":{},"f:uid":{}}}},"f:spec":{"f:output":{"f:to":{".":{},"f:kind":{},"f:name":{}}},"f:serviceAccount":{},"f:source":{"f:contextDir":{},"f:git":{".":{},"f:ref":{},"f:uri":{}},"f:type":{}},"f:strategy":{"f:sourceStrategy":{".":{},"f:env":{},"f:forcePull":{},"f:from":{".":{},"f:kind":{},"f:name":{}},"f:pullSecret":{".":{},"f:name":{}}},"f:type":{}},"f:triggeredBy":{}},"f:status":{"f:conditions":{".":{},"k:{\"type\":\"New\"}":{".":{},"f:lastTransitionTime":{},"f:lastUpdateTime":{},"f:status":{},"f:type":{}}},"f:config":{".":{},"f:kind":{},"f:name":{},"f:namespace":{}},"f:phase":{}}}}]},"spec":{"serviceAccount":"builder","source":{"type":"Git","git":{"uri":"https://github.com/jfclere/demo-webapp.git","ref":"master"},"contextDir":"/"},"strategy":{"type":"Source","sourceStrategy":{"from":{"kind":"DockerImage","name":"image-registry.openshift-image-registry.svc:5000/jfc/jboss-webserver54-tomcat9-openshift@sha256:75dcdf81011e113b8c8d0a40af32dc705851243baa13b68352706154174319e7"},"pullSecret":{"name":"builder-dockercfg-rvbh8"},"env":[{"name":"MAVEN_MIRROR_URL"},{"name":"ARTIFACT_DIR"}],"forcePull":true}},"output":{"to":{"kind":"ImageStreamTag","name":"test:latest"}},"resources":{},"postCommit":{},"nodeSelector":null,"triggeredBy":[{"message":"Generic WebHook","genericWebHook":{"secret":"\u003csecret\u003e"}}]},"status":{"phase":"New","config":{"kind":"BuildConfig","namespace":"jfc","name":"test"},"output":{},"conditions":[{"type":"New","status":"True","lastUpdateTime":"2020-10-30T12:29:30Z","lastTransitionTime":"2020-10-30T12:29:30Z"}]}}
    {
      "kind": "Status",
      "apiVersion": "v1",
      "metadata": {

      },
      "status": "Success",
      "message": "no git information found in payload, ignoring and continuing with build",
      "code": 200
    }
    ```

**Additional resources**

-   [Webhook
    Triggers](https://docs.openshift.com/container-platform/4.9/cicd/builds/triggering-builds-build-hooks.html#builds-webhook-triggers_triggering-builds-build-hooks)
-   [JWS Operator CRD parameters](#crd_parameters "Chapter 7. JWS Operator CRD parameters")

# Chapter 7. JWS Operator CRD parameters 

The JWS Operator provides a set of custom resource definition (CRD)
parameters. When you create a custom resource `WebServer` file
for a web application, you can specify parameter values in a
`<key>: <value>` format. The JWS Operator uses the information
that you specify in the custom resource `WebServer` file to
deploy the web application.

## 7.1. CRD parameter hierarchy 

The JWS Operator provides CRD parameters in the following hierarchical
format:

``` screen
applicationName: <value>
replicas: <value>
useSessionClustering: <value>
webImage:
   applicationImage: <value>
   imagePullSecret: <value>
   webApp:
      name: <value>
      sourceRepositoryURL: <value>
      sourceRepositoryRef: <value>
      contextDir: <value>
      webAppWarImage: <value>
      webAppWarImagePushSecret: <value>
      builder:
        image: <value>
        imagePullSecret: <value>
        applicationBuildScript: <value>
   webServerHealthCheck:
      serverReadinessScript: <value>
      serverLivenessScript: <value>
webImageStream:
   imageStreamName: <value>
   imageStreamNamespace: <value>
   webSources:
      sourceRepositoryUrl: <value>
      sourceRepositoryRef: <value>
      contextDir: <value>
      webSourcesParams:
         mavenMirrorUrl: <value>
         artifactDir: <value>
         genericWebHookSecret: <value>
         githubWebHookSecret: <value>
   webServerHealthCheck:
      serverReadinessScript: <value>
      serverLivenessScript: <value>
```

**Note:**
When you create a custom resource `WebServer` file, specify parameter names and values in the same hierarchical format that the preceding example outlines. For more information about creating a custom resource `WebServer` file, see [Deploying an existing JWS image](#deploy_existing_image "Chapter 4. Deploying an existing JWS image").

## 7.2. CRD parameter details 

The following table describes the CRD parameters that the JWS Operator
provides. This table shows each parameter name in the context of any
higher-level parameters that are above it in the hierarchy.

Description

`replicas`

The number of pods of the JBoss Web Server image that you want to run

For example:\
`replicas: 2`

`applicationName`

The name of the web application that you want the JWS Operator to deploy

The application name must be a unique value in the OpenShift namespace
or project. The JWS Operator uses the application name that you specify
to create the route to access the web application.

For example:\
`applicationName: my-app`

`useSessionClustering`

Enables DNSping session clustering

This is set to `false`by default. If you set this parameter to `true`, the image must be based on JBoss Web Server images, because session clustering uses the `ENV_FILES` environment variable and a shell script to add the clustering in the `server.xml` file.

**Note:**
In this release, the session clustering functionality is available as a
Technology Preview feature only. The current Operator version uses the
`DNS Membership Provider`, which is limited because of DNS
limitations. `InetAddress.getAllByName()` results are cached,
which means session replications might not work while scaling up.

For example:\
`useSessionClustering: true`

`webImage`

A set of parameters that controls how the JWS Operator deploys pods from
existing images

This parameter contains `applicationImage`, `imagePullSecret`, `webApp`, and`webServerHealthCheck` fields.

`webImage:`\
      `applicationImage`

The full path to the name of the application image that you want to
deploy

For example:\
`applicationImage: quay.io/$user/my-image-name`

`webImage:`\
      `imagePullSecret`

The name of the secret that the JWS Operator uses to pull images from
the repository

The secret must contain the key `.dockerconfigjson`. The JWS
Operator mounts the secret and uses it similar to
`--authfile /mount_point/.dockerconfigjson` to pull the images
from the repository.

The `Secret` object definition file might contain several
username and password values or tokens to allow access to images in the
image stream, the builder image, and images built by the JWS Operator.

For example:\
`imagePullSecret: mysecret`

`webImage:`\
      `webApp`

A set of parameters that describe how the JWS Operator builds the web
application that you want to add to the application image

If you do not specify the `webApp` parameter, the JWS Operator
deploys the web application without building the application.

This parameter contains `name`, `sourceRepositoryURL`, `sourceRepositoryRef`, `contextDir`, `webAppWarImage`, `webAppWarImagePushSecret`, and `builder` fields.

`webImage:\
      `webApp:`\
           `name`

The name of the web application file

The default name is `ROOT.war`.

For example:\
`name: my-app.war`

`webImage:`\
      `webApp:`\
           `sourceRepositoryURL`

The URL where the application source files are located

The source should contain a Maven `pom.xml` file to support a Maven build. When Maven generates a `.war` file for the application, the `.war` file is copied to the `webapps` directory of the image that the JWS Operator uses to deploy the application (for example, `/opt/jws-5.x/tomcat/webapps`).

For example:\
`sourceRepositoryUrl: 'https://github.com/$user/demo-webapp.git'`

`webImage:`\
      `webApp:`\
           `sourceRepositoryRef`

The branch of the source repository that the JWS Operator uses

For example:\
`sourceRepositoryRef: main`

`webImage:`\
      `webApp:`\
           `contextDir`

The subdirectory in the source repository where the `pom.xml`
file is located and the `mvn install` command is run

For example:\
`contextDir: /`

`webImage:`\
      `webApp:`\
           `webAppWarImage`

The URL of the images where the JWS Operator pushes the built image

`webImage:`\
      `webApp:`\
           `webAppWarImagePushSecret`

The name of the secret that the JWS Operator uses to push images to the
repository

The secret must contain the key `.dockerconfigjson`. The JWS Operator mounts the secret and uses it similar to `--authfile /mount_point/.dockerconfigjson` to push the image to the repository.

If the JWS Operator uses a pull secret to pull images from the
repository, you must specify the name of the pull secret as the value
for the `webAppWarImagePushSecret` parameter. See
[imagePullSecret](#pullsecret) for more information.

For example:\
`imagePullSecret: mysecret`

`webImage:`\
      `webApp:`\
           `builder`

A set of parameters that describe how the JWS Operator builds the web
application and creates and pushes the image to the image repository

**Note:**
To ensure that the builder can operate successfully and run commands
with different user IDs, the builder must have access to the
`anyuid` SCC (security context constraint). To grant the
builder access to the `anyuid` SCC, enter the following
command:

`oc adm policy add-scc-to-user anyuid -z builder`

This parameter contains `image`, `imagePullSecret`,
and `applicationBuildScript` fields.

`webImage:`\
      `webApp:`\
           `builder:`\
                `image`

The image of the container where the JWS Operator builds the web
application

For example:\
`image: quay.io/$user/tomcat10-buildah`

`webImage:`\
      `webApp:`\
           `builder:`\
                `imagePullSecret`

The name of the secret (if specified) that the JWS Operator uses to pull
the builder image from the repository

The secret must contain the key `.dockerconfigjson`. The JWS Operator mounts the secret and uses it similar to `--authfile /mount_point/.dockerconfigjson` to pull the images from the repository.

The `Secret` object definition file might contain several username and password values or tokens to allow access to images in the image stream, the builder image, and images built by the JWS Operator.

For example:\
`imagePullSecret: mysecret`

`webImage:`\
      `webApp:`\
           `builder:`\
                `applicationBuildScript`

The script that the builder image uses to build the application
`.war` file and move it to the `/mnt` directory

If you do not specify a value for this parameter, the builder image uses
a default script that uses Maven and Buildah.

`webImage:`\
      `webServerHealthCheck`

The health check that the JWS Operator uses

The default behavior is to use the health valve, which does not require
any parameters.

This parameter contains `serverReadinessScript` and
`serverLivenessScript` fields.

`webImage:`\
      `webServerHealthCheck:`\
           `serverReadinessScript`

A string that specifies the logic for the pod readiness health check

If this parameter is not specified, the JWS Operator uses the default
health check by using the OpenShift internal registry to check
`http://localhost:8080/health`.

For example:\
`serverReadinessScript: /bin/bash -c " /usr/bin/curl --noproxy '*' -s 'http://localhost:8080/health' | /usr/bin/grep -i 'status.*UP'"`

`webImage:`\
      `webServerHealthCheck:`\
           `serverLivenessScript`

A string that specifies the logic for the pod liveness health check

This parameter is optional.

`webImageStream`

A set of parameters that control how the JWS Operator uses an image
stream that provides images to run or to build upon

The JWS Operator uses the latest image in the image stream.

This parameter contains `applicationImage`, `imagePullSecret`, `webApp`, and `webServerHealthCheck` fields.

`webImageStream:`\
      `imageStreamName`

The name of the image stream that you have created to allow the JWS
Operator to find the base images

For example:\
`imageStreamName: my-image-name-imagestream:latest`

`webImageStream:`\
      `imageStreamNamespace`

The namespace or project where you have created the image stream

For example:\
`imageStreamNamespace: my-namespace`

`webImageStream:`\
      `webSources`

A set of parameters that describe where the application source files are
located and how to build them

If you do not specify the `webSources` parameter, the JWS
Operator deploys the latest image in the image stream.

This parameter contains `sourceRepositoryUrl`, `sourceRepositoryRef`, `contextDir`, and `webSourcesParams` fields.

`webImageStream:`\
      `webSources:`\
           `sourceRepositoryUrl`

The URL where the application source files are located

The source should contain a Maven `pom.xml` file to support a Maven build. When Maven generates a `.war` file for the application, the `.war` file is copied to the `webapps` directory of the image that the JWS Operator uses to deploy the application (for example, `/opt/jws-5.x/tomcat/webapps`).

For example:\
`sourceRepositoryUrl: 'https://github.com/$user/demo-webapp.git'`

`webImageStream:`\
      `webSources:`\
           `sourceRepositoryRef`

The branch of the source repository that the JWS Operator uses

For example:\
`sourceRepositoryRef: main`

`webImageStream:`\
      `webSources:`\
           `contextDir`

The subdirectory in the source repository where the `pom.xml`
file is located and the `mvn install` command is run

For example:\
`contextDir: /`

`webImageStream:`\
      `webSources:`\
           `webSourcesParams`

A set of parameters that describe how to build the application images

This parameter is optional.

This parameter contains `mavenMirrorUrl`, `artifactDir`, `genericWebHookSecret`, and `githubWebHookSecret` fields.

`webImageStream:`\
      `webSources:`\
           `webSourcesParams:`\
                `mavenMirrorUrl`

The Maven proxy URL that Maven uses to build the web application

This parameter is required if the cluster does not have internet access.

`webImageStream:`\
      `webSources:`\
           `webSourcesParams:`\
                `artifactDir`

The directory where Maven stores the `.war` file that Maven
generates for the web application

The contents of this directory are copied to the `webapps`
directory of the image that the JWS Operator uses to deploy the
application (for example, `/opt/jws-5.x/tomcat/webapps`).

The default value is `target`.

`webImageStream:`\
      `webSources:`\
           `webSourcesParams:`\
                `genericWebHookSecret`

The name of a secret for a generic webhook that can trigger a build

For more information about creating a secret, see [Creating a secret for a generic or GitHub webhook](#create_webhook_secret "Chapter 6. Creating a secret for a generic or GitHub webhook").

For more information about using generic webhooks, see [Webhook Triggers](https://docs.openshift.com/container-platform/4.9/cicd/builds/triggering-builds-build-hooks.html#builds-webhook-triggers_triggering-builds-build-hooks).

For example:\
`genericWebHookSecret: my-secret`

`webImageStream:`\
      `webSources:`\
           `webSourcesParams:`\
                `githubWebHookSecret`

The name of a secret for a GitHub webhook that can trigger a build

For more information about creating a secret, see [Creating a secret for a generic or GitHub webhook](#create_webhook_secret "Chapter 6. Creating a secret for a generic or GitHub webhook").

For more information about using GitHub webhooks, see [Webhook
Triggers](https://docs.openshift.com/container-platform/4.9/cicd/builds/triggering-builds-build-hooks.html#builds-webhook-triggers_triggering-builds-build-hooks).

**Note:**
You cannot perform manual tests of a GitHub webhook. GitHub generates
the payload and it is not empty.

`webImageStream:`\
      `webServerHealthCheck`

The health check that the JWS Operator uses

The default behavior is to use the health valve, which does not require
any parameters.

This parameter contains `serverReadinessScript` and
`serverLivenessScript` fields.

`webImageStream:`\
      `webServerHealthCheck:`\
           `serverReadinessScript`

A string that specifies the logic for the pod readiness health check

If this parameter is not specified, the JWS Operator uses the default health check by using the OpenShift internal registry to check `http://localhost:8080/health`.

For example:\
`serverReadinessScript: /bin/bash -c " /usr/bin/curl --noproxy '*' -s 'http://localhost:8080/health' | /usr/bin/grep -i 'status.*UP'"`

`webImageStream:`\
      `webServerHealthCheck:`\
           `serverLivenessScript`

A string that specifies the logic for the pod liveness health check

This parameter is optional.