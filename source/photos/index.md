---
title: 我的相册
date: 2026-06-29 10:00:00
type: photos
comments: false # 相册页关闭评论
fancybox: true
---
<style>
/* 相册分类封面网格样式，适配Next黑白主题 */
.album-container {
  width: 100%;
  margin: 2rem 0;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 24px;
}
.album-card {
  border-radius: 12px;
  overflow: hidden;
  background: #f7f7f7;
  transition: all 0.3s ease;
}
[data-theme="dark"] .album-card {
  background: #2a2a2a;
}
.album-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
}
.album-card a {
  text-decoration: none;
}
.album-cover {
  width: 100%;
  height: 180px;
  object-fit: cover;
  display: block;
}
.album-info {
  padding: 14px 16px;
  text-align: center;
}
.album-name {
  font-size: 16px;
  font-weight: 500;
  color: #333;
  margin: 0;
}
[data-theme="dark"] .album-name {
  color: #eee;
}
.album-desc {
  font-size: 13px;
  color: #888;
  margin: 4px 0 0;
}
</style>

<center style="margin-bottom:32px;font-size:18px;color:#666;">光影记录，留存日常与旅途</center>

<div class="album-container">
  <!-- 相册1：旅行相册，href对应子相册路径，src填Cloudflare R2图床封面图 -->
  <div class="album-card">
    <a href="/photos/travel/">
      <img class="album-cover" src="/images/album/cover.jpg" alt="旅行相册封面" loading="lazy">
      <div class="album-info">
        <h3 class="album-name">旅途风景</h3>
        <p class="album-desc">各地旅行风光合集</p>
      </div>
    </a>
  </div>

  <!-- 相册2：日常随拍 -->
  <div class="album-card">
    <a href="/photos/plants/">
      <img class="album-cover" src="/images/album/cover.jpg" alt="植物随拍封面" loading="lazy">
      <div class="album-info">
        <h3 class="album-name">植物随拍</h3>
        <p class="album-desc">花草树木随手记录</p>
      </div>
    </a>
  </div>

  <!-- 新增相册直接复制上面区块，修改href、图片链接、名称描述即可 -->
</div>