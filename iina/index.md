# IINA配置youtube-dl


&lt;!--more--&gt;



&gt; [IINA](https://github.com/iina/iina) 作为一款开源的 Swift 编写的播放器, 我愿称之为 mac 最好用的播放器



## 安装IINA

IINA的安装可以点开上面的地址进行安装. 



## 安装youtube-dl

```bash
&gt; brew install youtube-dl
```



安装好之后可以也可以把`ffmpeg`安装一下

```bash
&gt; brew install ffmpeg
```



### 配置IINA

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/20230421094636.png&#34; width = &#34;85%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       配置IINA   	&lt;/div&gt; &lt;/center&gt;

* youtube-dl路径: `/usr/local/bin`

* 额外参数: `format=&#34;bestvideo[height&lt;=?480]&#43;bestaudio/best[height&lt;=?480]&#34;`

  参数意义为播放视频分辨率



### 安装IINA浏览器插件

[Open In IINA - Chrome 应用商店 (google.com)](https://chrome.google.com/webstore/detail/open-in-iina/pdnojahnhpgmdhjdhgphgdcecehkbhfo)


---

> 作者:   
> URL: https://buli-home.cn/iina/  

