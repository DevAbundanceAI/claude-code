FROM node:20
ARG TZ
ENV TZ="$TZ"
ARG CLAUDE_CODE_VERSION=latest

# Install basic development tools + supervisor
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  jq \
  nano \
  vim \
  wget \
  libwebsockets17 \
  supervisor \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install ttyd from binary
RUN wget -O /usr/local/bin/ttyd https://github.com/tsl0922/ttyd/releases/download/1.7.4/ttyd.x86_64 \
  && chmod +x /usr/local/bin/ttyd

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

# Write supervisord config
RUN mkdir -p /etc/supervisor/conf.d && printf '%s\n' \
  '[supervisord]' \
  'nodaemon=true' \
  'logfile=/data/supervisord.log' \
  'logfile_maxbytes=10MB' \
  'pidfile=/tmp/supervisord.pid' \
  'user=root' \
  '' \
  '[program:ttyd]' \
  'command=ttyd -p 7681 -W tmux new-session -A -s main' \
  'autostart=true' \
  'autorestart=true' \
  'startsecs=3' \
  'startretries=999' \
  'stderr_logfile=/data/ttyd.err.log' \
  'stdout_logfile=/data/ttyd.out.log' \
  'user=root' \
  > /etc/supervisor/conf.d/supervisord.conf

ARG USERNAME=node

# Persist bash history
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

ENV DEVCONTAINER=true

# Create config directories and persistent data dir
RUN mkdir -p /home/node/.claude /data && \
  chown -R node:node /home/node/.claude /data

WORKDIR /data

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

USER node

ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin
ENV SHELL=/bin/zsh
ENV EDITOR=nano
ENV VISUAL=nano

ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install Claude Code
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}

USER root

EXPOSE 7681

CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
