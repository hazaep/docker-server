FROM n8nio/runners:latest
USER root
# Corepack + pnpm tiene problemas de store/PATH en esta imagen base.
# JS dependencies descartadas por ahora — solo Python.
# RUN cd /opt/runners/task-runner-javascript && npm install moment uuid
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas PyUp-LifeUp-API
COPY n8n-task-runners.json /etc/n8n-task-runners.json
USER runner
