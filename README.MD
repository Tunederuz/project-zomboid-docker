# Credits
Main project from https://github.com/Danixu/project-zomboid-server-docker

Dev branch support from https://github.com/Diagounal/project-zomboid-server-docker

# Project Zomboid dedicated server container

This Dockerfile will pre-configure Project Zombie dedicated server using SteamCMD.

The ready to use image is available here:

https://hub.docker.com/r/thunderzstrike/project-zomboid

> **Note:** If you want to play **Build 42** multiplayer, you must set the `STEAMAPPBRANCH` [environment variable](#environment-variables) to `unstable`.

**WARNING:** Running the image on Windows using WSL2 in Docker Desktop will make the server startup times significantly slower. This is fine unless you are running alot of mods in which case server startup times may vary from 15 - 40 minutes. The best way to solve this problem is to run your container on a Linux based system or in a Linux VM.

## Environment variables

This dockerfile converts some env variables to arguments in the server command, allowing to configure it. The image supports the following environment variables:

**Required variables**

`ADMINPASSWORD`: Sets or changes the admin password. Is mandatory at first startup or will fail. As the server logs every arguments in plain text, it's up to you to remove it or not after first run.

**Optional variables**

`ADMINUSERNAME`: Sets the administrator username. 
>Default: `admin`.

`PASSWORD`: Sets or changes server password.

`CACHEDIR`: Set the folder where the save data will be stored. You'll usually want to store that in a persistant volume. 
>Default: `/home/steam/Zomboid`

`DEBUG`: Enable the debug mode in server.

`FORCEUPDATE`: Force a server update on every start. Updates main game and workshop items.

`FORCESTEAMCLIENTSOUPDATE`: Forcibly update the steamclient.so libraries in the Project Zomboid installation directory before starting the server. May help with Workshop item downloads. Only if you encounter issues.

`IP`: Set the interface IP where the server will listen. It will listen to all interfaces if omitted.

`MEMORY`: Amount of memory to use in the server JVM. 
>Default: `8192m` (8GB)

`MODFOLDERS`: Mod folders to use to load the game mods. Allowed options are workshop, steam, and mods. Options must be separated by comma without spaces.

`NOSTEAM`: Starts the server in NoSteam mode, allowing to non steam users to connect.

`PORT`: Set the server port. 
>Default: `16261`

`SERVERNAME`: Set the server name, which allows you to have multiple server versions. Changing the name will create a new server data, so current progress will not be available. Also will not be lost, just reverting the server name change will load it again. **NOTE: Don't use spaces or special characters or admin user will fail**
> Default: `pz-dedicated-server`

`SERVERPRESET`: Set the new servers default preset. Allowed options are Apocalypse, Beginner, Builder, FirstWeek, SixMonthsLater, Survival and Survivor. 
>Default: `Apocalypse`

`SERVERPRESETREPLACE`: Replace the server preset by the one set by the SERVERPRESET variable. By default false to avoid to replace custom configurations once the server has started.

`SOFTRESET`: Performs a soft reset in the server. More info in the Soft Reset section.

`STEAMPORT1` & `STEAMPORT2`: Sets the two additional ports required for steam to work. I guess that those ports are UDP, but it can be also TCP.

`STEAMVAC`: Enables or disables de SteamVac protection in server.

`WORKSHOP_IDS`: Semicolon separated list of Workshop IDs to install to the server. Leaving this empty will clear any existing value!

`MOD_IDS`: Semicolon separated list of Mod IDs to install to the server.

`LANG`: This env variable will set the container locales, which will be used by the pz server to set the language. Example (en_EN.UTF-8, es_ES.UTF-8...).

`STEAMAPPBRANCH`: Sets the Steam beta branch to download (e.g., `unstable`). If not set, the stable branch is used.

Mods will install on second server start

Maps are added on third server start, after mods are added

## Soft Reset

Soft reset will perform the following tasks:

* Removes items from the ground.
* Removes items from containers and replaces them with new loot. This includes player-made containers.
* Removes corpses and zombified players.
* Resets zombies.
* Removes blood splatter.
* Resets building alarms.
* Resets the game clock. I’m not positive if this resets to day 1, but the original blog post implies that it does. So this would bring water and electricity back. Assuming your server settings have them * available at day 1.
* Player-made buildings won’t be deleted.
* Player inventories won’t reset.

Removing the following files in the Server data folder (ToDo: I have to search the equivalent files in modern version):

* all zombie_X_Y.bin files
* map_t.bin
* map_meta.bin
* reanimated.bin

Currently is not working correctly and will cause the server to crash when it is active. Also the above points are not performed so nothign will change in the server.

## Required ports

The required ports are docummented in the [Official wiki](https://pzwiki.net/wiki/Dedicated_Server#Forwarding_Required_Ports)

For now the ports are the following, but it can change.

### Steam clients ports

This ports are used by the Steam clients to connect with the server, so it must be opened in firewall and configured in NAT.

* 8766 UDP
* 8767 UDP
* 16261 UDP (configurable)

### Non Steam clients ports

This ports are used by the No Steam clients to connect with the server, so it must be opened in firewall and configured in NAT.

* 8766 UDP
* 8767 UDP
* 16261 UDP (configurable)
* 16262 - 16272 TCP (range depends of the above port and the clients slots)

### Notes

* The 16261 port can be changed using the PORT environment variable. This will affect at the TCP ports range, changing it to the next port. For example, if PORT is set to 12234, the TCP range will be 12234 - 12244.
* The TCP ports range is used by the clients, so every client slot will need a port. If your server plans to host 50 clients, then you will need to open 50 ports in firewall/nat... (16262 - 16312).
* The Wiki says that Non Steam ports is supposed to be for non steam clients, but I am using the steam client and those ports are used too.

## How to use the image

To setup the server you can use the docker-compose program:

* Copy the .env.template to .env and edit it with the settings you want. The only mandatory variable is ADMINPASSWORD and just the first time the server is executed (once is started at least once, it can be removed). The other variables are optional.
* Run the `docker-compose up -d` command to start the server.

## Scripts folder

In the scripts folders there are three scripts:

* entry.sh: The script used in the image to start the server
* search_folder.sh: Script to search through mod folders for any maps 
