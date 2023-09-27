Certainly! Here's an example `deployment.yaml` file that defines three init containers to copy files into the `/tmp/agent` folder, and then a normal container that has access to the same folder as a common volume mount:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: init-containers-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: init-containers-demo
  template:
    metadata:
      labels:
        app: init-containers-demo
    spec:
      initContainers:
        - name: init-container-1
          image: busybox
          command:
            [
              "sh",
              "-c",
              "echo 'This is init container 1' > /tmp/agent/file1.txt",
            ]
          volumeMounts:
            - name: agent-volume
              mountPath: /tmp/agent
        - name: init-container-2
          image: busybox
          command:
            [
              "sh",
              "-c",
              "echo 'This is init container 2' > /tmp/agent/file2.txt",
            ]
          volumeMounts:
            - name: agent-volume
              mountPath: /tmp/agent
        - name: init-container-3
          image: busybox
          command:
            [
              "sh",
              "-c",
              "echo 'This is init container 3' > /tmp/agent/file3.txt",
            ]
          volumeMounts:
            - name: agent-volume
              mountPath: /tmp/agent
      containers:
        - name: main-container
          image: your-main-container-image
          volumeMounts:
            - name: agent-volume
              mountPath: /tmp/agent
      volumes:
        - name: agent-volume
          emptyDir: {}
```

In this `deployment.yaml` file:

- Three init containers (`init-container-1`, `init-container-2`, and `init-container-3`) are defined. Each of them uses the BusyBox image to run a shell command that creates a file in the `/tmp/agent` directory. They all share a common volume mount named `agent-volume`.

- The main container (`main-container`) is your normal application container, and it also mounts the `agent-volume` volume at `/tmp/agent`. This allows the main container to access the files created by the init containers.

Make sure to replace `your-main-container-image` with the actual image you want to use for your main application container.
