# 🔹 Authentication & Authorization in OpenShift (Defaults)

## 1. **Default Users**

When a fresh OpenShift cluster is installed:

* **`kubeadmin`**

  * Bootstrap superuser created during installation.
  * Credentials stored in a secret:

    ```bash
    oc get secret kubeadmin -n kube-system -o jsonpath='{.data.kubeadmin-password}' | base64 -d
    ```
  * Intended for **initial login & setup only**.
  * Best practice: replace with IDP users and delete it later.

* **System Users (`system:*`)**

  * Automatically created, not real human users.
  * Examples:

    * `system:admin` → superuser account, bypasses RBAC.
    * `system:unauthenticated` → represents requests without login.
    * `system:authenticated` → represents all logged-in users.
    * `system:serviceaccount:<namespace>:<sa-name>` → service accounts used by workloads.

---

## 2. **Default Roles**

Roles are RBAC definitions that control access. Two scopes:

* **ClusterRoles (cluster-wide):**

  * `cluster-admin` → Full control across the cluster.
  * `admin` → Project-level admin (manage roles, resources).
  * `edit` → Create/update/delete most resources in a project.
  * `view` → Read-only access.
  * `self-provisioner` → Allows creating new projects (`oc new-project`).
  * Many **system roles** exist for internal components (e.g., `system:image-puller`, `system:deployer`, `system:node`, `system:router`, etc.).

* **Roles (namespace-scoped):**

  * Created automatically per project.
  * Bindings like `admin`, `edit`, `view` are namespaced when applied to a project.

---

## 3. **Default Role Bindings**

Some ClusterRoleBindings are created automatically:

* **`cluster-admin`** → bound to `system:admin` and bootstrap users (like `kubeadmin`).
* **`self-provisioner`** → by default bound to `system:authenticated:oauth` (all logged-in users), allowing everyone to create projects.
  ⚠️ In production, admins often **remove this binding**:

  ```bash
  oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
  ```
* **Service Account bindings**:

  * `system:deployer` → to deployment SA.
  * `system:image-builder` → to build SA.
  * `system:image-puller` → to default SA in all namespaces.
  * `system:router`, `system:registry` → bound to their respective SAs.

---

## 4. **Identity Providers (IDPs)**

* By default, OpenShift uses only `kubeadmin`.
* To enable real user login, you must configure one or more **IDPs** in `oauth/cluster`:

  * LDAP/AD
  * GitHub
  * GitLab
  * Google
  * HTPasswd
  * OpenID Connect (Keycloak, etc.)

Example (HTPasswd):

```bash
oc create secret generic htpass-secret --from-file=htpasswd=/path/to/users.htpasswd -n openshift-config

oc apply -f - <<EOF
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: local
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
EOF
```

---

## 5. **Service Accounts (SAs)**

* Automatically created per project:

  * `default` → used by pods without a specified SA.
  * `deployer`, `builder` → used internally by deployment & build controllers.
* SAs can be granted roles like normal users:

  ```bash
  oc adm policy add-role-to-user edit system:serviceaccount:myproject:default
  ```

---

## 6. **Token Authentication**

* Users and SAs authenticate via tokens.
* Tokens are stored in secrets (for SAs).
* Example: get a SA token:

  ```bash
  oc sa get-token default -n myproject
  ```

---

# ✅ Quick Summary Table

| **Component**        | **Default Provided**                                                     | **Purpose**                       |
| -------------------- | ------------------------------------------------------------------------ | --------------------------------- |
| **Users**            | `kubeadmin`, `system:*`, service accounts                                | Initial admin + system identities |
| **ClusterRoles**     | `cluster-admin`, `admin`, `edit`, `view`, `self-provisioner`, `system:*` | Predefined RBAC roles             |
| **RoleBindings**     | `self-provisioner` → all authenticated users (default), SA role bindings | Controls access automatically     |
| **IDP**              | None (must configure)                                                    | Real user authentication          |
| **Service Accounts** | `default`, `builder`, `deployer` per project                             | Pod/workload identities           |

---

👉 So out of the box:

* You log in with **`kubeadmin`**.
* All logged-in users (if IDP is added) can **create projects** (`self-provisioner`).
* Projects come with **default service accounts and bindings**.

---


# Here’s how you can **safely delete the default `kubeadmin` account** from the cluster:

---

## 🔹 1. Make sure another admin user exists

Before deleting, confirm you have a valid user with **`cluster-admin`** privileges. Example:

```bash
oc adm policy add-cluster-role-to-user cluster-admin myadminuser
```

Verify:

```bash
oc whoami
oc get clusterrolebindings | grep myadminuser
```

---

## 🔹 2. Delete the `kubeadmin` secret

The `kubeadmin` credentials are stored in a secret in the `kube-system` namespace.
To delete it:

```bash
oc delete secret kubeadmin -n kube-system
```

---

## 🔹 3. Verify removal

Now if someone tries to log in with `kubeadmin`:

```bash
oc login -u kubeadmin -p <old-password> https://api.<cluster-domain>:6443
```

…it will **fail** because the secret no longer exists.

---

## 🔹 4. Best Practice

* Do this only **after**:

  * At least one other user (via IDP) is a `cluster-admin`.
  * You’ve confirmed you can log in with that user.
* Once deleted, `kubeadmin` **cannot be restored automatically**; you’d need to manually recreate a secret to bring it back.

---

✅ **Summary:**

* Grant `cluster-admin` to a real user.
* Delete the secret:

  ```bash
  oc delete secret kubeadmin -n kube-system
  ```
* `kubeadmin` login will stop working.

---

Perfect 👍 — let’s go step by step on how to **manually recreate the `kubeadmin` account** in OpenShift if you’ve deleted it.

⚠️ **Note:** The `kubeadmin` user is special — it only works because OpenShift’s authentication operator knows about it during installation. Once deleted, you can simulate it by creating a **new secret** with the same name, but Red Hat officially recommends using an IDP + `cluster-admin` users instead. Still, here’s how to bring it back:

---

## 🔹 1. Generate a password

Pick a strong password or generate one:

```bash
openssl rand -base64 16
```

Say the password is:

```
RedHat@123
```

---

## 🔹 2. Create the `kubeadmin` secret

Recreate the secret in the `kube-system` namespace:

```bash
oc create secret generic kubeadmin \
  -n kube-system \
  --from-literal=kubeadmin=RedHat@123
```

---

## 🔹 3. (Optional) Grant `cluster-admin` role

Just to be sure, explicitly bind it (in most cases, OpenShift already treats `kubeadmin` as cluster-admin if the secret exists):

```bash
oc adm policy add-cluster-role-to-user cluster-admin kubeadmin
```

---

## 🔹 4. Login with kubeadmin

Now you should be able to log in again:

```bash
oc login -u kubeadmin -p RedHat@123 https://api.<cluster-domain>:6443
```

---

## 🔹 5. Verify permissions

Check if it has cluster-admin rights:

```bash
oc whoami
oc auth can-i '*' '*' --all-namespaces
```

You should see:

```
yes
```

---

✅ **Summary:**

* Create a new secret:

  ```bash
  oc create secret generic kubeadmin -n kube-system --from-literal=kubeadmin=<password>
  ```
* Bind cluster-admin if needed:

  ```bash
  oc adm policy add-cluster-role-to-user cluster-admin kubeadmin
  ```
* Log in with the new password.

---

