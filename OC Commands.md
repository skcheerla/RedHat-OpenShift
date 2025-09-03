The `oc` command-line interface (CLI) is a powerful tool for interacting with an OpenShift cluster. It's built on top of the Kubernetes `kubectl` CLI but adds a number of OpenShift-specific features for application development, building, and deployment.

Here's a breakdown of common `oc` command categories and some of the most useful commands within them:

### General and Project Management

* **`oc login`**: Log in to an OpenShift cluster. You can specify a server URL, username, and password.
* **`oc logout`**: Log out of the current session.
* **`oc new-project <project-name>`**: Create a new project (namespace).
* **`oc project <project-name>`**: Switch to a different project.
* **`oc projects`**: List all projects you have access to.
* **`oc status`**: Show an overview of the current project's resources, including running pods, services, and routes.
* **`oc whoami`**: Display information about the currently logged-in user.

### Resource Management (similar to `kubectl`)

* **`oc get <resource-type>`**: List one or more resources. For example, `oc get pods`, `oc get deployments`, `oc get svc`.
* **`oc describe <resource-type>/<resource-name>`**: Show detailed information about a specific resource.
* **`oc create -f <file-name>`**: Create a resource from a YAML or JSON file.
* **`oc apply -f <file-name>`**: Apply a configuration to a resource, creating it if it doesn't exist or updating it if it does.
* **`oc delete <resource-type>/<resource-name>`**: Delete a resource.
* **`oc edit <resource-type>/<resource-name>`**: Open a text editor to modify a resource's configuration directly.
* **`oc expose <resource-type>/<resource-name>`**: Create a route to expose a service externally.

### Application and Deployment

* **`oc new-app`**: Create a new application from source code, an existing image, or a template.
    * `oc new-app .` (from local Git repo)
    * `oc new-app <remote-git-url>`
    * `oc new-app <image-name>`
* **`oc rollout`**: Manage a deployment's rollout.
    * **`oc rollout status <deployment-name>`**: Check the status of a new deployment.
    * **`oc rollout history <deployment-name>`**: View the history of deployments.
    * **`oc rollout undo <deployment-name>`**: Roll back to a previous deployment.
* **`oc scale <resource-type>/<resource-name> --replicas=<number>`**: Change the number of running pods for a resource.
* **`oc autoscale <resource-type>/<resource-name>`**: Set up horizontal pod autoscaling.

### Builds and Images

* **`oc new-build`**: Create a new build configuration from a source repository.
* **`oc start-build <build-config-name>`**: Manually start a build.
* **`oc cancel-build <build-name>`**: Cancel a running or pending build.
* **`oc import-image <image-stream-name>`**: Import the latest image information from a Docker registry.
* **`oc tag`**: Tag existing images into image streams.

### Troubleshooting and Debugging

* **`oc logs <pod-name>`**: Print the logs for a pod. Use `-f` to follow the logs in real-time.
* **`oc rsh <pod-name>`**: Start a remote shell session inside a pod.
* **`oc exec <pod-name> -- <command>`**: Execute a command inside a container within a pod.
* **`oc cp <local-path> <pod-name>:<remote-path>`**: Copy files from the local filesystem to a pod.
* **`oc port-forward <pod-name> <local-port>:<remote-port>`**: Forward a local port to a port on a pod.
* **`oc debug <resource-type>/<resource-name>`**: Create a debug pod for troubleshooting.

### Administrator Commands (`oc adm`)

These commands are typically for cluster administrators.

* **`oc adm must-gather`**: Collect diagnostic information about the cluster.
* **`oc adm policy`**: Manage authorization policies, roles, and users.
* **`oc adm prune`**: Prune old builds, images, or deployments.

For a comprehensive list of commands and their options, you can use the built-in help:

* **`oc --help`**: Get a list of all top-level commands.
* **`oc <command> --help`**: Get help for a specific command, e.g., `oc new-app --help`.
* 





1️⃣ Log in to the OpenShift internal registry
	1.	Get your registry URL:

oc get route default-route -n openshift-image-registry

	•	Usually it’s like default-route-openshift-image-registry.apps.<cluster-domain>

	2.	Log in using podman or docker (depending on your local setup):

docker login -u $(oc whoami) -p $(oc whoami -t) <registry-url>

	•	oc whoami -t generates your OpenShift token automatically.

⸻

2️⃣ Tag your local image for the OCP registry

Suppose you built your image locally:

docker build -t nginx-app:updated .

Tag it for OpenShift registry:

docker tag nginx-app:updated <registry-url>/<project>/<image-name>:<tag>

Example:

docker tag nginx-app:updated default-route-openshift-image-registry.apps.ocp.example.com/myproject/nginx-app:v2

	•	<project> → your OpenShift project/namespace
	•	<tag> → version (e.g., v2)

⸻

3️⃣ Push the image to OpenShift registry

docker push <registry-url>/<project>/<image-name>:<tag>

	•	OpenShift internal registry will store the image.

⸻

4️⃣ Update your Deployment in OpenShift

Point your deployment to the new image:

oc set image deployment/<deployment-name> nginx=<registry-url>/<project>/<image-name>:<tag>

	•	Check rollout:

oc rollout status deployment/<deployment-name>
oc get pods


