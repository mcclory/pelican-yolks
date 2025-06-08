############################################################
# Dockerfile that contains SteamCMD
############################################################
FROM harbor.ops.mcclory.io/games/steam-cmd-root:latest AS build_stage

LABEL maintainer="patrick@mcclory.io"
ARG PUID=1000

ENV TARGET_USER=container 
ENV TARGET_HOMEDIR="/home/container"
ENV STEAMCMDDIR="/home/container/steamcmd"

RUN set -x && \
	# Install, update & upgrade packages
	apt-get update && \
	apt-get install -y --no-install-recommends --no-install-suggests lib32stdc++6 lib32gcc-s1 ca-certificates nano curl locales && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
	dpkg-reconfigure --frontend=noninteractive locales && \
    dpkg --add-architecture i386 && \
    apt update && \
    apt upgrade -y && \
    apt install -y --no-install-recommends --no-install-suggests \
        curl \
        g++ \
        gcc \
        gdb \
        iproute2 \
        locales \
        net-tools \
        netcat-traditional \
        tar \
        telnet \
        tini \
        tzdata \
        wget \
        lib32gcc-s1 \
        lib32stdc++6 \
        lib32tinfo6 \
        lib32z1 \
        libcurl3-gnutls:i386 \
        libcurl4-gnutls-dev:i386 \
        libcurl4:i386 \
        libfontconfig1 \
        libgcc-11-dev \
        libgcc-12-dev \
        libncurses5:i386 \
        libsdl1.2debian \
        libsdl2-2.0-0:i386 \
        libssl-dev:i386 \
        libtinfo6:i386 && \
	# Create unprivileged user
    mkdir -p ${STEAMCMDDIR} && \
    chown -R "${TARGET_USER}":"${TARGET_USER}" "${TARGET_HOMEDIR}" && \
	# Download SteamCMD, execute as user
    su "${TARGET_USER}" -c \
		"mkdir -p \"${STEAMCMDDIR}\" \
                && curl -fsSL 'https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz' | tar xvzf - -C \"${STEAMCMDDIR}\" \
                && \"${STEAMCMDDIR}/steamcmd.sh\" +quit \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${STEAMCMDDIR}/steamservice.so\" \
                && mkdir -p \"${TARGET_HOMEDIR}/.steam/sdk32\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${TARGET_HOMEDIR}/.steam/sdk32/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamcmd\" \"${STEAMCMDDIR}/linux32/steam\" \
                && mkdir -p \"${TARGET_HOMEDIR}/.steam/sdk64\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamclient.so\" \"${TARGET_HOMEDIR}/.steam/sdk64/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamcmd\" \"${STEAMCMDDIR}/linux64/steam\" \
                && ln -s \"${STEAMCMDDIR}/steamcmd.sh\" \"${STEAMCMDDIR}/steam.sh\"" && \
	# Symlink steamclient.so; So misconfigured dedicated servers can find it
 	ln -s "${STEAMCMDDIR}/linux64/steamclient.so" "/usr/lib/x86_64-linux-gnu/steamclient.so" && \
    apt-get clean autoclean && \
    apt-get autoremove -y && \
	rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*  

# Install rcon
RUN cd /tmp/ && \
    curl -sSL https://github.com/gorcon/rcon-cli/releases/download/v0.10.3/rcon-0.10.3-amd64_linux.tar.gz > rcon.tar.gz && \
    tar xvf rcon.tar.gz && \
    mv rcon-0.10.3-amd64_linux/rcon /usr/local/bin/

FROM build_stage AS source-root

USER        container
ENV         USER=container HOME=/home/container
WORKDIR ${STEAMCMDDIR}

FROM source-root AS source
# Switch to user
USER container

FROM source-root AS csgo
ENV STEAM_USER=anonymous \
    STEAM_PASS="" \
    STEAM_AUTH="" \
    SRCDS_APPID=740 \
    SRCDS_BETAID="" \
    SRCDS_BETAPASS="" \
    HLDS_GAME="csgo" \
    INSTALL_FLAGS="" \
    VALIDATE=0 \
    WINDOWS_INSTALL=0

RUN /home/container/steamcmd/steamcmd.sh +force_install_dir /home/container +login ${STEAM_USER} ${STEAM_PASS} ${STEAM_AUTH} +app_update 1007 +app_update ${SRCDS_APPID} $( [[ -z ${HLDS_GAME} ]] || printf %s "+app_set_config 90 mod ${HLDS_GAME}" )  ${INSTALL_FLAGS} $( [[ "${VALIDATE}" == "1" ]] && printf %s 'validate' ) +quit

ENV SRCDS_FPSMAX=300 \
	SRCDS_TICKRATE=128 \
	SRCDS_PORT=27015 \
	SRCDS_TV_PORT=27020 \
	SRCDS_CLIENT_PORT=27005 \
	SRCDS_NET_PUBLIC_ADDRESS="0" \
	SRCDS_IP="0" \
	SRCDS_LAN="0" \
	SRCDS_MAXPLAYERS=14 \
	SRCDS_TOKEN=0 \
	SRCDS_RCONPW="changeme" \
	SRCDS_PW="changeme" \
	SRCDS_STARTMAP="de_dust2" \
	SRCDS_REGION=3 \
	SRCDS_MAPGROUP="mg_active" \
	SRCDS_GAMETYPE=0 \
	SRCDS_GAMEMODE=1 \
	SRCDS_HOSTNAME="New \"${STEAMAPP}\" Server" \
	SRCDS_WORKSHOP_START_MAP=0 \
	SRCDS_HOST_WORKSHOP_COLLECTION=0 \
	SRCDS_WORKSHOP_AUTHKEY="" \
	ADDITIONAL_ARGS=""

COPY etc/csgo "/etc/csgo"


COPY --chown=container:container etc/entrypoint.sh /entrypoint.sh

USER ${TARGET_USER}

CMD ["/bin/bash", "/entry.sh"]

FROM csgo AS source-metamod

ENV METAMOD_VERSION=1.12

FROM csgo AS source-sourcemod

ENV SOURCEMOD_VERSION=1.12
