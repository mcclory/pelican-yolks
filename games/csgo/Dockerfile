

FROM harbor.ops.mcclory.io/games/base/source:latest AS csgo

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

COPY --chown=container:container ./entrypoint.sh /entrypoint.sh

USER ${TARGET_USER}

CMD ["/bin/bash", "/entrypoint.sh"]

FROM csgo AS csgo-metamod

ENV STEAMAPPDIR="/home/container" \
    STEAMAPP="csgo" \
    METAMOD_VERSION=1.12

RUN LATESTMM=$(wget -qO- https://mms.alliedmods.net/mmsdrop/${METAMOD_VERSION}/mmsource-latest-linux) && \
    wget -qO- https://mms.alliedmods.net/mmsdrop/${METAMOD_VERSION}/${LATESTMM} | tar xvzf - -C "${STEAMAPPDIR}/${STEAMAPP}"	

FROM csgo-metamod AS csgo-sourcemod

ENV ENV STEAMAPPDIR="/home/container" \
    STEAMAPP="csgo" \
    SOURCEMOD_VERSION=1.12 
    
RUN LATESTSM=$(wget -qO- https://sm.alliedmods.net/smdrop/${SOURCEMOD_VERSION}/sourcemod-latest-linux) 
