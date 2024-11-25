ARG VARIANT=12-buster
FROM node:${VARIANT}

#
# Handle the ssh key.
#

# Set up the environment
ENV HOME=/root
WORKDIR /root

# Ensure necessary tools are installed
RUN apt-get update && apt-get install -y \
    openssh-client \
    && rm -rf /var/lib/apt/lists/*

# Ensure proper ownership and permissions for .bashrc
USER root
RUN touch /root/.bashrc && chmod 644 /root/.bashrc && chown root:root /root/.bashrc

# Create the .ssh directory and set permissions
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh

# Add the SSH config file
RUN echo "Host github.com-neverendingsupport\n\
    Hostname github.com\n\
    StrictHostKeyChecking no\n\
    User AndreAngelantoni\n\
    AddKeysToAgent yes\n\
    IdentityFile ~/.ssh/id_rsa_no_passphrase\n\
    IdentitiesOnly yes" > $HOME/.ssh/config && chmod 600 $HOME/.ssh/config

# Add the private key file
COPY id_rsa_no_passphrase $HOME/.ssh/id_rsa_no_passphrase
RUN chmod 600 $HOME/.ssh/id_rsa_no_passphrase

# Test SSH setup (optional)
RUN ssh -T git@github.com || echo "SSH setup successful"


#
# Install Node.
#

ARG USERNAME=node
ARG NPM_GLOBAL=/usr/local/share/npm-global

USER root
RUN useradd -m vscode

# Add NPM global to PATH.
ENV PATH=${NPM_GLOBAL}/bin:${PATH}

RUN \
    # Configure global npm install location, use group to adapt to UID/GID changes
    if ! cat /etc/group | grep -e "^npm:" > /dev/null 2>&1; then groupadd -r npm; fi \
    && usermod -a -G npm ${USERNAME} \
    && umask 0002 \
    && mkdir -p ${NPM_GLOBAL} \
    && touch /usr/local/etc/npmrc \
    && chown ${USERNAME}:npm ${NPM_GLOBAL} /usr/local/etc/npmrc \
    && chmod g+s ${NPM_GLOBAL} \
    && npm config -g set prefix ${NPM_GLOBAL} \
    && su ${USERNAME} -c "npm config -g set prefix ${NPM_GLOBAL}" \
    # Install eslint
    && su ${USERNAME} -c "umask 0002 && npm install -g eslint" \
    && npm cache clean --force > /dev/null 2>&1

# Install extra apps.
RUN apt-get update && apt-get install -y git tree jq && apt-get clean

# Install gulp globally.
RUN npm install -g gulp

# Install app's node dependencies.
RUN npm install

# Set up quality of life improvements. Add to all users.
RUN echo "alias ll='ls -la'" >> /etc/bash.bashrc

RUN mkdir -p /workspaces && chown -R vscode:vscode /workspaces

USER vscode

CMD ["bash", "-c", "cd /workspaces && exec /bin/bash"]

#
# For reference.
#
# -------------------------------------------------------------

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment if you want to install an additional version of node using nvm
# ARG EXTRA_NODE_VERSION=10
# RUN su node -c "source /usr/local/share/nvm/nvm.sh && nvm install ${EXTRA_NODE_VERSION}"

# [Optional] Uncomment if you want to install more global node modules
# RUN su node -c "npm install -g <your-package-list-here>"