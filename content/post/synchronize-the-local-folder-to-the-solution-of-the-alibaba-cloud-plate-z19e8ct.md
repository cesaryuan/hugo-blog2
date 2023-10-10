<h1>准备工作</h1>
<ul>
<li>
<p>同步软件：GoodSync<br />
这里我选择的是GoodSync，目前主流的此类同步软件我知道的有以下几种</p>
<blockquote>
<table>
<thead>
<tr>
<th>官网</th>
<th>特色（个人愚见）</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://www.goodsync.com/cn">GoodSync</a></td>
<td>优点：功能最全面，UI比较好<br />缺点：同步文件不支持忽略文件时间只比较文件大小</td>
</tr>
<tr>
<td><a href="https://freefilesync.org/download.php">FreeFileSync</a></td>
<td>优点：免费<br />缺点：不支持同步到WebDav</td>
</tr>
<tr>
<td><a href="http://cn.filegee.com/product.html">FileGee</a></td>
<td>优点：国产，支持百度网盘/Dropbox/OneDrive/阿里云OSS/AmazonS3<br />缺点：UI较丑，扫描完成后没有同步树的展示</td>
</tr>
<tr>
<td><a href="https://www.odrive.com/homepage5b">odrive</a></td>
<td>未使用</td>
</tr>
<tr>
<td><a href="http://www.verysync.com/">微力同步</a></td>
<td>简单看了下好像不符合我的需求，</td>
</tr>
<tr>
<td>syncthing</td>
<td></td>
</tr>
<tr>
<td>nextcloud</td>
<td></td>
</tr>
</tbody>
</table>
</blockquote>
<p>经过一些考虑，我选择了GoodSync</p>
</li>
<li>
<p>将阿里云盘进行处理以使其支持GoodSync的软件：<a href="https://github.com/alist-org/alist">Alist</a></p>
<p>这里我没有使用 <a href="https://github.com/messense/aliyundrive-webdav">messense/aliyundrive-webdav: 阿里云盘 WebDAV 服务</a>，因为我扫了一眼Release，发现没有exe（狗头）</p>
<p>不使用CloudDrive的原因是CloudDrive会丢失文件的时间信息</p>
</li>
<li>
<p>云服务：阿里云盘</p>
<p>可能有同学说为什么不使用OneDrive或者其他云服务，因为我觉得阿里云盘目前访问速度比较快</p>
</li>
</ul>
<h1>同步方法</h1>
<ol>
<li>
<p>首先使用Alist将阿里云盘映射为WebDav服务</p>
<p>具体步骤见Alist官网</p>
</li>
<li>
<p>在GoodSync中将文件夹同步到webdav</p>
<p>这里要注意有一个坑是必须关闭图中的选项，不然同步到</p>
<p><img src="https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202310102144093.png" alt="image.png" /></p>
</li>
</ol>
<p>简单的小分享，大家也可以分享自己的方案</p>
