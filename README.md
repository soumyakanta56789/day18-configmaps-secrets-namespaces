# day18-configmaps-secrets-namespaces
## Kubernetes Core Concepts Q&A

### 1. What is the difference between a ConfigMap and a Secret?
* **ConfigMap:** Used to store non-sensitive configuration data (like application configuration parameters, environment variables, or config files) in plain text.
* **Secret:** Designed specifically to store sensitive data (like passwords, API keys, SSH keys, or database credentials). While both inject data into Pods similarly, Secrets are handled with extra care by Kubernetes (e.g., they are stored in memory (`tmpfs`) on nodes rather than written to disk).

### 2. Why must referenced ConfigMaps and Secrets exist in the Pod's namespace?
Namespaces act as strict security and administrative boundaries. A Pod is isolated to its own namespace and cannot reach across boundaries to read data from another namespace. Allowing a Pod to reference ConfigMaps or Secrets in other namespaces would break multi-tenant security isolation.

### 3. What is the difference between environment variables and mounted files?
* **Environment Variables:** Data is loaded directly into the container's process environment at startup. It is lightweight and easy for applications to read, but the value is static for the life of that container process.
* **Mounted Files:** Data is exposed inside the container as actual files on a virtual directory path. This is ideal for larger configuration sets (like an `index.html` or a complex `.conf` file).

### 4. Why does changing a ConfigMap not update an existing container's environment automatically?
Environment variables are injected into a container's main process **only at boot time**. Once the process starts running inside Linux, its environment block cannot be dynamically updated from the outside. If you update a ConfigMap mapped to environment variables, you must restart or roll out the Pod to inject the new values. *(Note: ConfigMaps mounted as files do auto-update over time, but environment variables never do).*

### 5. Why is base64 not sufficient to protect a Secret?
Base64 is an **encoding algorithm, not an encryption method**. It simply translates text into a different readable format to handle binary data safely. Anyone who can view a base64 string can instantly decode it back to plain text using standard commands (like `echo "encoded-string" | base64 --decode`). Real protection requires access controls (RBAC) and data encryption-at-rest.

### 6. What can cause `CreateContainerConfigError`?
This error means the Pod's scheduling succeeded, but Kubernetes failed to build the container's runtime setup. The most common causes are:
* A referenced **ConfigMap** does not exist.
* A referenced **Secret** does not exist.
* The specific **key** inside the ConfigMap or Secret (like `password` or `APP_ENV`) is missing or misspelled in your YAML.

### 7. How do namespaces help separate dev and prod resources?
Namespaces create virtual logical clusters within the same physical Kubernetes cluster. They provide:
* **Name Isolation:** You can have a deployment named `web` and a secret named `db-secret` in both namespaces simultaneously without name collisions.
* **Access Control:** You can limit developer access so they can only modify resources inside `dev` while leaving `prod` locked down.
* **Resource Quotas:** You can restrict how much CPU and memory the `dev` environment can consume so it never starves `prod` resources.
