## Adding extra dependencies[](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-task-runners#adding-extra-dependencies)

### 1. Extend the `n8nio/runners` image[](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-task-runners#id-1-extend-the-n8niorunners-image)

You can extend the `n8nio/runners` image to add extra dependencies to the runners. You'll need `n8nio/runners:1.121.0` or later to do this.

Copy

```
FROM n8nio/runners:1.121.0
USER root
RUN cd /opt/runners/task-runner-javascript && pnpm add moment uuid
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas
COPY n8n-task-runners.json /etc/n8n-task-runners.json
USER runner
```

You must also allowlist any first-party or third-party packages for use by the Code node. Do this by editing the configuration file `n8n-task-runners.json` to include the packages in your extended image.

Copy

```
{
  "task-runners": [
    {
      "runner-type": "javascript",
      "env-overrides": {
        "NODE_FUNCTION_ALLOW_BUILTIN": "crypto",         // <-- allowlist Node.js builtin modules here
        "NODE_FUNCTION_ALLOW_EXTERNAL": "moment,uuid",   // <-- allowlist third-party JS packages here
      }
    },
    {
      "runner-type": "python",
      "env-overrides": {
        "PYTHONPATH": "/opt/runners/task-runner-python",
        "N8N_RUNNERS_STDLIB_ALLOW": "json",              // <-- allowlist Python standard library packages here
        "N8N_RUNNERS_EXTERNAL_ALLOW": "numpy,pandas"     // <-- allowlist third-party Python packages here
      }
    }
  ]
}
```

- `NODE_FUNCTION_ALLOW_BUILTIN`: comma-separated list of allowed node builtin modules.
    
- `NODE_FUNCTION_ALLOW_EXTERNAL`: comma-separated list of allowed JS packages.
    
- `N8N_RUNNERS_STDLIB_ALLOW`: comma-separated list of allowed Python standard library packages.
    
- `N8N_RUNNERS_EXTERNAL_ALLOW`: comma-separated list of allowed Python packages.
    

### 2. Build your custom image[](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-task-runners#id-2-build-your-custom-image)

For example, from the n8n repository root:

Copy

```
docker buildx build \
  -f docker/images/runners/Dockerfile \
  -t n8nio/runners:custom \
  .
```

### 3. Run the image[](https://docs.n8n.io/deploy/host-n8n/configure-n8n/set-up-task-runners#id-3-run-the-image)

For example:

Copy

```
docker run --rm -it \
  -e N8N_RUNNERS_AUTH_TOKEN=test \
  -e N8N_RUNNERS_LAUNCHER_LOG_LEVEL=debug \
  -e N8N_RUNNERS_TASK_BROKER_URI=http://host.docker.internal:5679 \
  -p 5680:5680 \
  n8nio/runners:custom
```

[](https://docs.n8n.io/deploy/host-n8n/configure-n8n/manage-settings-using-environment-variables)