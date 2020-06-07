# Scale multiple Jibri instance with Docker

## 1. Requirements

- Any linux distrobution with Alsa/Pulseaudio support.
- Proper Internet access.
- Docker installed.
- Required kernel module for playback interfaces
- **Valid SSL certificate** for jitsi-web instance. 



### 1.1 Install required module for the kernel

Keep in mind, after installation maybe required to **<u>reboot</u>**.

```
# install the module
apt update && apt install linux-image-extra-virtual
# configure 5 capture/playback interfaces
echo "options snd-aloop enable=1,1,1,1,1 index=0,1,2,3,4" > /etc/modprobe.d/alsa-loopback.conf
# setup autoload the module
echo "snd-aloop" >> /etc/modules
# check that the module is loaded
lsmod | grep snd_aloop
```



## 2. Deploy jitsi-meet stack.

**First deploy whole jitsi meet based on official documentation, it mean you should have working version of jitsi-meet stack with ability to record at least one session.**

## 3. Scale Jibri instances

you can scale container of Jibri with these command.

**Let's assume we want to have 4 concurrent recording**

```shell
docker-compose -f docker-compose.yml -f jibri.yml  up -d --scale jibri=4
```

Then you will see something like this:

```shell
root@test:~/docker-jitsi-meet# docker-compose -f docker-compose.yml -f jibri.yml  up -d --scale jibri=4
dockerjitsimeet_prosody_1 is up-to-date
dockerjitsimeet_web_1 is up-to-date
dockerjitsimeet_jicofo_1 is up-to-date
dockerjitsimeet_jvb_1 is up-to-date
Starting dockerjitsimeet_jibri_1 ... done
Creating dockerjitsimeet_jibri_2 ...
Creating dockerjitsimeet_jibri_3 ...
Creating dockerjitsimeet_jibri_4 ...
Creating dockerjitsimeet_jibri_2 ... done
Creating dockerjitsimeet_jibri_3 ... done
Creating dockerjitsimeet_jibri_4 ... done
```

Then you should **change the loopback device in `/home/jibri/.asoundrc` **

```shell
docker exec -t dockerjitsimeet_jibri_2 sed -i 's/Loopback/2/g' /home/jibri/.asoundrc
docker exec -t dockerjitsimeet_jibri_3 sed -i 's/Loopback/3/g' /home/jibri/.asoundrc
docker exec -t dockerjitsimeet_jibri_4 sed -i 's/Loopback/4/g' /home/jibri/.asoundrc
```

Now restart the containers:

```shell
docker stop dockerjitsimeet_jibri_{2,3,4}
docker start dockerjitsimeet_jibri_{2,3,4}
```

