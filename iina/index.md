# IINA配置youtube-dl


<!--more-->



> [IINA](https://github.com/iina/iina) 作为一款开源的 Swift 编写的播放器, 我愿称之为 mac 最好用的播放器



## 安装IINA

IINA的安装可以点开上面的地址进行安装. 



## 安装youtube-dl

```bash
> brew install youtube-dl
```



安装好之后可以也可以把`ffmpeg`安装一下

```bash
> brew install ffmpeg
```



### 配置IINA

<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/20230421094636.png" width = "85%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       配置IINA   	</div> </center>

* youtube-dl路径: `/usr/local/bin`

* 额外参数: `format="bestvideo[height<=?480]+bestaudio/best[height<=?480]"`

  参数意义为播放视频分辨率



### 安装IINA浏览器插件

[Open In IINA - Chrome 应用商店 (google.com)](https://chrome.google.com/webstore/detail/open-in-iina/pdnojahnhpgmdhjdhgphgdcecehkbhfo)

