#ARG VARIANT=12-buster
#ARG VARIANT=16-buster
#ARG VARIANT=18-buster
ARG VARIANT=22-buster
FROM node:${VARIANT}

# Set up the environment
ENV HOME=/root
WORKDIR /root
USER root

# Install openssh-client.
RUN apt-get update && apt-get install -y \
    openssh-client \
    && rm -rf /var/lib/apt/lists/*

# Install sudo.
RUN apt-get update && apt-get install -y sudo && \
echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Install extra tools
RUN apt-get update && apt-get install -y git tree jq && apt-get clean

# Set up quality of life improvements for everyone.
RUN echo "alias ll='ls -la'" >> /etc/bash.bashrc && \
    echo "export PATH=/usr/local/share/npm-global/bin:\$PATH" >> /etc/bash.bashrc && \
    echo "alias ll='ls -la'" >> /etc/profile && \
    echo "export PATH=/usr/local/share/npm-global/bin:\$PATH" >> /etc/profile

# Change prompt to user:dirctory$
RUN echo "export PS1='\u@\h:\w\$ '" >> /etc/bash.bashrc && \
echo "export PS1='\u@\h:\w\$ '" >> /etc/profile


#
# Install Node, pnpm and gulp.
#
ARG USERNAME=node
ARG NPM_GLOBAL=/usr/local/share/npm-global

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

# [Optional] Uncomment if you want to install an additional version of node using nvm.
# ARG EXTRA_NODE_VERSION=10
# RUN su node -c "source /usr/local/share/nvm/nvm.sh && nvm install ${EXTRA_NODE_VERSION}"

# Install pnpm globally.
RUN npm install -g pnpm

# Install gulp globally.
RUN npm install -g gulp

#
# Create user(s).
#
RUN useradd -m -s /bin/bash vscode

#
# Set up ssh.
#

# Create the .ssh directory and set permissions.
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh

# Add the SSH config file
RUN echo "Host github.com-neverendingsupport\n\
    Hostname github.com\n\
    StrictHostKeyChecking no\n\
    User AndreAngelantoni\n\
    AddKeysToAgent yes\n\
    IdentityFile ~/.ssh/id_rsa_no_passphrase\n\
    IdentitiesOnly yes" > $HOME/.ssh/config && chmod 600 $HOME/.ssh/config

# Copy the public key.
COPY .devcontainer/id_rsa_no_passphrase $HOME/.ssh/id_rsa_no_passphrase
RUN chmod 600 $HOME/.ssh/id_rsa_no_passphrase

# Test SSH setup (optional).
RUN ssh -T git@github.com || echo "SSH setup successful"

#
# Set up a workspace directory, then log in with a clean shell.
#

# Create a workspace directory for the vscode user.
RUN mkdir -p /workspace && chown -R vscode:vscode /workspace

# Add a .bash_profile to user vscode.
RUN echo "source /etc/profile" > /home/vscode/.bash_profile && \
    chown vscode:vscode /home/vscode/.bash_profile

# Copy all files above .devcontainer (from the context)
# into /workspace. Change ownership before changing user.
COPY . /workspace
RUN chown -R vscode:vscode /workspace

USER vscode
WORKDIR /workspace

#
# API server runs on port 3000.
# Documentation server runs on port 4321.
#
EXPOSE 3000
EXPOSE 4321

# Prevents looking for the .bash_profile in /root.
ENV HOME=/home/vscode
CMD ["/bin/bash", "--login"]