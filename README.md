# Music streamer

While running we use our phone with Youtube.
This leads to 2 issues when we do not want to pay for premium option.

1. We can not lock the phone
2. Too much ads, which is a pain when running as we have to skip or listen to them 
3. Data exposure

Here we propose an alternative with VLC stream

Pre-requisite of tha do is: https://github.com/scoulomb/myDNS/blob/master/2-advanced-bind/5-real-own-dns-application/6-use-linux-nameserver-part-0.md

## Stream audio

### Deploy VLC server

- Get vlc
- Configure the server


```
mkddir zik; cd zik
vlc *.mp3 --random --sout '#transcode{acodec=vorb,ab=128}:standard{access=http,mux=ogg,dst=:20001}' -I ncurses

sudo ufw allow 20001
```

**Source**: http://blog.kewix.fr/index.php/post/2009/07/02/Faire-du-streaming-de-ses-MP3-vers-un-client-distant-avec-VLC


### Configure the Android Client

- Playstore -> VLC
- Add stream 

```
# If on LAN
192.168.1.32:20001

```

To find your machine IP address for example do

```
$ ip addr | grep eno1 | grep inet | awk {'print $2'}
192.168.1.32/24
```

Note we can stream the content on Google home

### Configure a reverse NAT

Use-case: phone with 4G while not at home

Make your stream available publicly by configuring NAT in your box

It is in In http://192.168.1.1/network/nat

```
vlc	TCP	Port	20001	192.168.1.32
```

Application is now available via public IP on the internet:

```shell script
<box public ip> 109.29.148.109:20001
```

### Use DNS configuration performed in prerequisite with a smart switch to avoid to go outside to come back inside when on same LAN

Note we could use directly: `scoulomb.ddns.net`. but no switching and could use 2 DNS one private and one public-->


#### Example using Gandi namserver

We perform configuration here:
https://github.com/scoulomb/myDNS/blob/master/2-advanced-bind/5-real-own-dns-application/6-use-linux-nameserver-part-a.md#abstract-dynamic-dns-via-a-cname

- with a CNAME in Gandi 

````shell script
home 300 IN CNAME scoulomb.ddns.net.
````

- And in SFR

````shell script
192.168.1.32	home.coulombel.it/site
````



#### Example with own nameserver

We perform configuration here: 
https://github.com/scoulomb/myDNS/blob/master/2-advanced-bind/5-real-own-dns-application/6-use-linux-nameserver-part-b.md#achieve-with-our-own-dns-the-same-service-performed-by-gandi-for-application-deployed-in-section-part-a-to-abstract-dynamic-dns-name

- with a CNAME in our nameserver accessible from NAT with IT registrar pointing to our nameserver.

````shell script
home 300 IN CNAME scoulomb.ddns.net.
````

- And in SFR

````shell script
192.168.1.32	home.coulombel.it/site
````

#### Tests

We tested successfully this with our own DNS nameserver here (with registrar) using IT domain and Gandi live DNS using site domain.

We used this protocol to test switch occurred use `home.coulombel.it/site:20001`. Use NAT config: http://192.168.1.1/network/nat
- Enable NAT when phone is on 4G, click on button with Antenna it will work
- Disable NAT when phone is on 4G, click on button with Antenna (otherwise stream continue) it will not work
=> it uses public IP
- Keep Disable NAT when phone is on Wifi, click on button with Antenna (wait) it should continue to work unlike 4G.
=> it uses private IP
- Set NAT back (no need to test NAT and private IP)
=> tested OK


Rather than define a DNS record we could also use a Gandi redirection to abstract the port.
(not tested), but need external DNS  (so internet)




## Get content 

You can difitalize your CD, vinyl

We can also use youtube-dl.

Their GIT repo is down: https://github.com/ytdl-org/youtube-dl due to DCMA.
Howver here: https://youtube-dl.org/
We can get repo content in a tarball.
It is interesting to have the full readme, otherwise we can follow the procedure below.


```
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/y
sudo chmod a+rx /usr/local/bin/y
```
If you have this error

```
y
/usr/bin/env: ‘python’: No such file or directory
```


You should do

```
alias ytdl='/usr/bin/python3 /usr/local/bin/y'
ytdl https://www.youtube.com/watch?v=L-5M1_DKvb0&list=RDMML-5M1_DKvb0&start_radio=1 
```

**Source**: https://askubuntu.com/questions/1037666/youtube-dl-python-not-found-18-04


I advise to donwload MP3 only (to reduce stream data consumption)

To downlaoad mp3 only, create a youtube dl config.


```
mkdir -p ~/.config/youtube-dl/ 
touch ~/.config/youtube-dl/config 

echo '--extract-audio
--audio-format mp3' > ~/.config/youtube-dl/config 
```

**Source**: https://doc.ubuntu-fr.org/youtube-dl

If you have following error

```
ytdl https://www.youtube.com/watch?v=L-5M1_DKvb0&list=RDMML-5M1_DKvb0&start_radio=1 

ERROR: ffprobe/avprobe and ffmpeg/avconv not found. Please install one.
```

Do


```
sudo apt-get update
sudo apt-get install ffmpeg
```


Then download content

```
cd zik


ytdl https://www.youtube.com/watch?v=L-5M1_DKvb0&list=RDMML-5M1_DKvb0&start_radio=1 
ytdl https://www.youtube.com/watch?v=fCZVL_8D048
ytdl https://www.youtube.com/watch?v=3OhqqT49VS0
```

Note we can do a batch

```
ytdl https://www.youtube.com/watch?v=wjaJRSuSQww https://www.youtube.com/watch?v=hrAoU2O8sfE https://www.youtube.com/watch?v=LN7fBVPbvWk
ytdl https://www.youtube.com/watch?v=ddjTNiCKBBU https://www.youtube.com/watch?v=rywUS-ohqeE
```

### Alternative

- Play MP3 wiht Jupyter: https://stackoverflow.com/questions/33417151/playing-mp3-in-a-folder-with-jupyter-notebook
- Use HTML5 music player: https://github.com/MoePlayer/APlayer

### Misc notes

#### Music stopping at end of each track

We should check the `no-loop` option.

**Source**: https://superuser.com/questions/1304188/how-to-prevent-automatic-jumping-to-next-track#:~:text=With%20VLC%20you%20can%20stop,no%2Dloop%20option%20is%20selected


#### Start server

Before run we need to start VLC server.
It could be started remotely via Jupyter or SSH (NAT again).
Cf. https://github.com/scoulomb/myDNS/blob/master/2-advanced-bind/5-real-own-dns-application/6-use-linux-nameserver-part-a.md


#### Optional TODO/DONE

- check data conso: 80 mb for 1.5 h of stream from Android
- dockerize and run in kube: decided to not do
- https://www.omgubuntu.co.uk/2018/02/vlc-3-0-chromecast-support-new-features

