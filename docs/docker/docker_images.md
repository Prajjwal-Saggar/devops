# Docker Images

1. **Docker Images** are built using a `Dockerfile`.
2. A **Dockerfile** contains step-by-step instructions required to assemble an image.
3. Once an image is created, it can be distributed or shared. Every time an **Image** is executed, a new isolated **Container** is instantiated.
4. **Relationship:** Dockerfile $\rightarrow$ Docker Image $\rightarrow$ Docker Container.

---

> [!NOTE]
> * **Pull an Image:** `docker pull <image_name>`
> * **Run an Image:** `docker run <image_name>`
> * **Detached Mode:** `docker run -d <image_name>`
>   * The `-d` flag runs the container in **detached mode** (in the background), freeing up your current terminal prompt.

---

## How to Work with a Dockerfile

1. A `Dockerfile` defines the exact runtime environment for your custom application so it can be containerized.
2. It is a plain text file (named `Dockerfile` without an extension) containing specific Docker directives (`FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`).

### General Development Workflow
1. **Clone the repository** from GitHub/GitLab to your local workspace.
2. **Navigate** to the project root directory where the `Dockerfile` is located.
3. **Build the image:**
   ```bash
   docker build -t example_name .
   ```
4. **Verify the image creation:**
   ```bash
   docker images
   ```
5. **Run the container:**
   ```bash
   docker run -d example_name
   ```

---

## Is the `docker build` command correct?

**Yes, the command `docker build -t example_name .` is completely correct.**

### What does the `-t` flag do?

* **`-t` stands for "Tag":** It assigns a human-readable **name** (and optionally a version tag) to your built image so you don't have to refer to it by a random, complex Image ID.
* **Format:** `-t <repository_name>:<tag>`
  * If you run `docker build -t my-app .`, Docker defaults the tag to `latest` (`my-app:latest`).
  * You can specify custom versions: `docker build -t my-app:v1.0 .`

### What does the `.` at the end mean?
The period `.` specifies the **build context** directory (usually the current working directory). Docker looks inside this directory for the `Dockerfile` and uses it as the root path for any file copying operations (`COPY . .`).


---

## Dockerfile Syntax

```dockerfile
# Step 1: Load the environment runtime requirement (Alpine is a lightweight OS distribution)
FROM openjdk:17-jdk-alpine

# Step 2: Create and set the working directory inside the container
WORKDIR /app

# Step 3: Copy the source code from the host machine to the container
COPY src/Main.java /app/Main.java
# Note: "COPY . ." can also be used to copy all files from the context directory

# Step 4: Compile the Java application source file
RUN javac Main.java

# Step 5: Define the default execution command when the container launches
CMD ["java", "Main"]
```

---

### Key Takeaways & Directives Breakdown

* **`FROM`**: Sets the base image (e.g., `openjdk:17-jdk-alpine` provides a minimal Java 17 Development Kit environment).
* **`WORKDIR`**: Sets the default working directory for subsequent instructions (`COPY`, `RUN`, `CMD`).
* **`COPY`**: Transfers files/directories from your local host machine into the image filesystem.
* **`RUN`**: Executes commands **during the build process** to prepare the image layer (e.g., installing packages, compiling code).
* **`CMD`**: Specifies the **default command executed when the container starts**.

> [!NOTE]
> **Build Instructions vs. Runtime Executables:**
> * Instructions like `RUN`, `COPY`, and `WORKDIR` execute sequentially during `docker build` to construct image layers.
> * `CMD` does **not** execute during image build time. Instead, it gets embedded into the image metadata as the default container entry command.
> * Arguments passed during `docker run` will completely override `CMD`.

---

### `CMD` vs. `ENTRYPOINT`

| Feature | `CMD` | `ENTRYPOINT` |
| :--- | :--- | :--- |
| **Primary Purpose** | Provides default command/arguments for execution. | Defines a fixed executable for the container. |
| **Runtime Override** | **Easily overridden** by adding commands at `docker run`. | **Hard to override** (requires explicit `--entrypoint` flag). |
| **Usage Pattern** | Best for flexible containers or optional CLI parameters. | Best when treating containers as single-purpose binaries. |
---


### What is a Port in Docker?

A port mapping connects your host machine's network port to a specific port inside the running Docker container, allowing external traffic to reach the application.

```bash
docker run -p 8080:80 flask-app
```

---

### Understanding Port Mapping (`-p host_port:container_port`)

* **First Port (`8080`): Host Port**  
  The port on your actual host machine (e.g., your laptop, EC2 instance, or server). This is the port you access in your browser: `http://localhost:8080`.

* **Second Port (`80`): Container Port**  
  The internal port where your application inside the container is listening (e.g., standard Flask or HTTP port).

---

> [!NOTE]
> **Key Directive:** You expose internal container ports inside your `Dockerfile` using the `EXPOSE` instruction (e.g., `EXPOSE 80`), and bind it to a host port at runtime using `-p <host_port>:<container_port>`.


> [!Docker exec]
> This command `docker exec -it` is when we want to use container mysql and terminal of mysql inside that container so we run the command `docker exec -it container_id bash`.

