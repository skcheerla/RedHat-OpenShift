Creating and using a template in Red Hat OpenShift is a two-step process: first, you define the template, and then you use it to deploy an application. Here is a step-by-step procedure for both with explanations.

### Step 1: Create the Application Template

The first step is to create a YAML or JSON file that defines the objects you want to deploy. This file will serve as your blueprint.

#### Procedure:

1.  **Define the `Template` object:** Start by defining the template's metadata and a description. This makes it easy to identify later.

    ```yaml
    apiVersion: template.openshift.io/v1
    kind: Template
    metadata:
      name: my-app-template
      annotations:
        description: "A simple template to deploy a basic web application."
    parameters:
    - name: APP_NAME
      description: "The name of the application."
      value: "simple-web-app"
    - name: DOCKER_IMAGE
      description: "The container image to use."
      value: "nginx:latest"
    - name: APP_REPLICAS
      description: "The number of pod replicas."
      value: "2"
    objects:
    - kind: DeploymentConfig
      apiVersion: apps.openshift.io/v1
      metadata:
        name: ${APP_NAME}
      spec:
        replicas: ${{APP_REPLICAS}}
        selector:
          app: ${APP_NAME}
        template:
          metadata:
            labels:
              app: ${APP_NAME}
          spec:
            containers:
            - name: ${APP_NAME}
              image: ${DOCKER_IMAGE}
              ports:
              - containerPort: 8080
    - kind: Service
      apiVersion: v1
      metadata:
        name: ${APP_NAME}
      spec:
        ports:
        - port: 8080
          targetPort: 8080
        selector:
          app: ${APP_NAME}
    - kind: Route
      apiVersion: route.openshift.io/v1
      metadata:
        name: ${APP_NAME}
      spec:
        to:
          kind: Service
          name: ${APP_NAME}
    ```

2.  **Add `parameters`:** In the `parameters` section, define the configurable variables. Each parameter should have a `name`, a `description`, and an optional `value` (default value). In the example above, `APP_NAME`, `DOCKER_IMAGE`, and `APP_REPLICAS` are parameters.

3.  **Include `objects`:** Under the `objects` section, list all the Kubernetes/OpenShift resources you want to create. This can include `DeploymentConfig`, `Service`, `Route`, etc. Use the `${PARAMETER_NAME}` syntax to reference the parameters you defined. For example, the `name` of the DeploymentConfig and Service is set to `${APP_NAME}`, making them dynamic.

4.  **Save the file:** Save the file with a `.yaml` or `.json` extension.

-----

### Step 2: Deploy the Application Using the Template

Once you have a template file, you can use it to deploy the application in your OpenShift project.

#### Procedure:

1.  **Log in to OpenShift:** Ensure you are logged into your OpenShift cluster via the `oc` command-line tool.

2.  **Process the template:** Use the `oc process` command to substitute the parameter values and generate the final YAML manifest. This is a crucial step for validation before you apply the resources.

    ```bash
    oc process -f my-app-template.yaml \
      -p APP_NAME=my-web-app \
      -p DOCKER_IMAGE=quay.io/openshift/hello-openshift:latest \
      -p APP_REPLICAS=3 \
      > processed-app.yaml
    ```

      * **`-f my-app-template.yaml`**: Specifies the template file to use.
      * **`-p PARAMETER_NAME=VALUE`**: Specifies the value for each parameter. You can skip parameters with a default value, or override them.
      * **`> processed-app.yaml`**: Redirects the processed output to a new YAML file. You can inspect this file to see the final configuration.

3.  **Apply the processed resources:** Use the `oc apply` command to create the resources in your project.

    ```bash
    oc apply -f processed-app.yaml
    ```

    This command will create a `DeploymentConfig`, a `Service`, and a `Route` with the names and configurations you specified via the parameters.

4.  **Alternatively, use a single command:** You can combine the processing and creation steps into a single command using a pipe.

    ```bash
    oc process -f my-app-template.yaml -p APP_NAME=my-web-app -p DOCKER_IMAGE=quay.io/openshift/hello-openshift:latest -p APP_REPLICAS=3 | oc apply -f -
    ```

This is a more direct and common way to deploy from a template. The `-f -` tells `oc apply` to read the input from the standard input (the output of `oc process`).

#### Explanation:

  * **Template Processing:** The `oc process` command is a powerful tool for template management. It takes the template file and the provided parameter values to generate a finalized manifest. This allows you to create a dynamic blueprint that can be used for different environments (e.g., development, staging, production) or applications by simply changing the parameter values.

  * **Instantiation:** When you `oc apply` the final manifest, OpenShift reads the file and creates all the defined resources, bringing your application to life. This automates the deployment of complex applications, ensuring consistency and repeatability across all your projects.
