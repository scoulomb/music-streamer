# Music streamer

While running we use our phone with Youtube.
This leads to 2 issues when we do not want to pay for premium option.

1. We can not lock the phone
2. Too much ads, which is a pain when running as we have to skip or listen to them 
3. Data exposure

Here we propose an alternative with VLC stream


## Stream audio

### Configure server 

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

### Make your stream availble publicly

Use-case: phone with 3G while not at home


As a prerequisute you need to configure a Dynamic DNS.
Procedure is described here: https://github.com/scoulomb/myDNS/blob/master/2-advanced-bind/5-real-own-dns-application/6-use-linux-nameserver-part-a.md#configure-a-dynamic-dns

- Configure NAT on your box

```
vlc	TCP	Port	20001	192.168.1.32
```
- Configure VLC stream on Android client

```
# If on 3G (public IP)
<your DDNS Name>.ddns.net:20001
```

This will work on LAN or not (when using 3G).

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

#### TODO

- check data conso


