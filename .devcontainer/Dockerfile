FROM debian:bookworm

LABEL org.opencontainers.image.authors="Eneru"
LABEL org.opencontainers.image.licenses="GPL-3.0-or-later"
LABEL org.opencontainers.image.title="GOAD devcontainer"
LABEL org.opencontainers.image.description="Image that embeds every tools needed to develop on the GOAD project, and to deploy it using terraform providers."
LABEL org.opencontainers.image.base.name="debian:bookworm"

SHELL ["/bin/sh", "-c"]

# Install packages available through apt
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    bash-completion=* \
    curl=* \
    git=* \
    gnupg2=* \
    iproute2=* \
    jq=* \
    lsb-release=* \
    locales=* \
    openvpn=* \
    openssh-client=* \
    python3 \
    python3-dev \
    python3-venv \
    python3-pip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV PATH="$HOME/.local/bin:$PATH"

# Install Powershell
RUN . /etc/os-release && \
    curl -o packages-microsoft-prod.deb "https://packages.microsoft.com/config/debian/$VERSION_ID/packages-microsoft-prod.deb" && \
    dpkg -i packages-microsoft-prod.deb && \
    rm -f packages-microsoft-prod.deb && \
    apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y powershell=* && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Terraform as root user
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -o - https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list && \
    apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y terraform=* && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
SHELL ["/bin/sh", "-c"]

# Get latest version of Hadolint
RUN curl --silent -o hadolint-Linux-x86_64 "https://github.com/hadolint/hadolint/releases/download/$(curl --silent 'https://api.github.com/repos/hadolint/hadolint/releases/latest' | jq -r .tag_name)/hadolint-Linux-x86_64" && \
    mv hadolint-Linux-x86_64 /usr/local/bin/hadolint && \
    chmod +x /usr/local/bin/hadolint

# Install pipx
RUN python3 -m pip install --break-system-packages --no-cache-dir pipx && python3 -m pipx ensurepath

# Install terraform auto-completions
RUN terraform -install-autocomplete

# Install openstack client
RUN pipx install openstacksdk python-openstackclient

# Install ansible, ansible auto-completions and ansible tools
RUN pipx install --include-deps ansible && \
    pipx inject --include-apps ansible argcomplete && activate-global-python-argcomplete && \
    pipx install ansible-dev-tools ansible-lint ansible-creator

# Install mkdocs material
RUN pipx install --include-deps mkdocs-material

# Prepare lang environment for ansible
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8

# Set lang environment variables for ansible
ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8"