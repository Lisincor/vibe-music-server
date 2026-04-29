# Vibe Music Server 接口文档

## 概述

本文档描述 Vibe Music 后端 API 接口。

**基础 URL**: `http://localhost:8080`

**认证方式**: 除公开接口外，所有接口需要在请求头中携带 JWT Token：
```
Authorization: Bearer <token>
```

**统一响应格式**:
```json
{
  "code": 0,      // 0=成功, 1=失败
  "message": "操作成功",
  "data": {}
}
```

**分页响应格式**:
```json
{
  "code": 0,
  "message": "操作成功",
  "data": {
    "total": 100,      // 总记录数
    "records": [...]   // 数据列表
  }
}
```

---

## 公开接口（无需认证）

### 轮播图

#### 获取轮播图列表
- **URL**: `GET /banner/getBannerList`
- **响应**:
```json
{
  "code": 0,
  "message": "操作成功",
  "data": [
    {
      "id": 1,
      "url": "https://minio.../banner.jpg",
      "status": 1
    }
  ]
}
```

### 歌手

#### 获取所有歌手
- **URL**: `POST /artist/getAllArtists`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "name": "周杰伦"
}
```
- **响应**: 分页歌手列表

#### 获取歌手详情
- **URL**: `GET /artist/getArtistDetail/{id}`
- **响应**: 歌手详细信息（包含该歌手的所有歌曲）

#### 获取随机歌手
- **URL**: `GET /artist/getRandomArtists`
- **响应**: 随机10位歌手列表

### 歌曲

#### 获取所有歌曲
- **URL**: `POST /song/getAllSongs`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "songName": "稻香",
  "artistName": "周杰伦"
}
```
- **响应**: 分页歌曲列表

#### 获取推荐歌曲
- **URL**: `GET /song/getRecommendedSongs`
- **响应**: 推荐歌曲列表（20首）

#### 获取歌曲详情
- **URL**: `GET /song/getSongDetail/{id}`
- **响应**: 歌曲详细信息

### 歌单

#### 获取所有歌单
- **URL**: `POST /playlist/getAllPlaylists`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "title": "华语"
}
```
- **响应**: 分页歌单列表

#### 获取推荐歌单
- **URL**: `GET /playlist/getRecommendedPlaylists`
- **响应**: 推荐歌单列表

#### 获取歌单详情
- **URL**: `GET /playlist/getPlaylistDetail/{id}`
- **响应**: 歌单详情（包含歌曲列表）

---

## 用户接口（需要用户 Token）

### 用户认证

#### 发送验证码
- **URL**: `GET /user/sendVerificationCode?email=user@example.com`
- **说明**: 发送6位验证码到指定邮箱

#### 用户注册
- **URL**: `POST /user/register`
- **请求体**:
```json
{
  "username": "music_fan",
  "password": "Pass123!",
  "email": "user@example.com",
  "verificationCode": "ABC123"
}
```
- **验证规则**:
  - 用户名: 4-16位字母、数字、下划线、连字符
  - 密码: 8-18位，需包含字母和数字
  - 验证码: 6位字母或数字

#### 用户登录
- **URL**: `POST /user/login`
- **请求体**:
```json
{
  "email": "user@example.com",
  "password": "Pass123!"
}
```
- **响应**:
```json
{
  "code": 0,
  "message": "操作成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### 用户登出
- **URL**: `POST /user/logout`
- **请求头**: `Authorization: Bearer <token>`

#### 重置密码
- **URL**: `PATCH /user/resetUserPassword`
- **请求体**:
```json
{
  "email": "user@example.com",
  "password": "NewPass123!",
  "verificationCode": "ABC123"
}
```

### 用户信息

#### 获取用户信息
- **URL**: `GET /user/getUserInfo`
- **响应**:
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "username": "music_fan",
    "email": "user@example.com",
    "avatar": "https://minio.../avatar.jpg",
    "createTime": "2025-01-01"
  }
}
```

#### 更新用户信息
- **URL**: `PUT /user/updateUserInfo`
- **请求体**:
```json
{
  "username": "new_name"
}
```

#### 更新用户头像
- **URL**: `PATCH /user/updateUserAvatar`
- **Content-Type**: `multipart/form-data`
- **参数**: `avatar` (文件)

#### 更新用户密码
- **URL**: `PATCH /user/updateUserPassword`
- **请求体**:
```json
{
  "oldPassword": "OldPass123!",
  "newPassword": "NewPass123!"
}
```

#### 注销账号
- **URL**: `DELETE /user/deleteAccount`

### 收藏管理

#### 获取收藏的歌曲
- **URL**: `POST /favorite/getFavoriteSongs`
- **请求体**: 同 `SongDTO`
- **响应**: 分页收藏歌曲列表

#### 收藏歌曲
- **URL**: `POST /favorite/collectSong?songId=1`

#### 取消收藏歌曲
- **URL**: `DELETE /favorite/cancelCollectSong?songId=1`

#### 获取收藏的歌单
- **URL**: `POST /favorite/getFavoritePlaylists`
- **请求体**: 同 `PlaylistDTO`
- **响应**: 分页收藏歌单列表

#### 收藏歌单
- **URL**: `POST /favorite/collectPlaylist?playlistId=1`

#### 取消收藏歌单
- **URL**: `DELETE /favorite/cancelCollectPlaylist?playlistId=1`

### 评论管理

#### 添加歌曲评论
- **URL**: `POST /comment/addSongComment`
- **请求体**:
```json
{
  "songId": 1,
  "content": "这首歌太好听了！"
}
```

#### 添加歌单评论
- **URL**: `POST /comment/addPlaylistComment`
- **请求体**:
```json
{
  "playlistId": 1,
  "content": "这个歌单很棒！"
}
```

#### 点赞评论
- **URL**: `PATCH /comment/likeComment/{id}`

#### 取消点赞评论
- **URL**: `PATCH /comment/cancelLikeComment/{id}`

#### 删除评论
- **URL**: `DELETE /comment/deleteComment/{id}`

### 反馈管理

#### 提交反馈
- **URL**: `POST /feedback/addFeedback?content=建议增加歌词显示功能`

---

## 管理端接口（需要管理员 Token）

### 管理员认证

#### 管理员注册
- **URL**: `POST /admin/register`
- **请求体**:
```json
{
  "username": "admin",
  "password": "Admin123!"
}
```

#### 管理员登录
- **URL**: `POST /admin/login`
- **请求体**: 同上
- **响应**: 包含 token

#### 管理员登出
- **URL**: `POST /admin/logout`

### 用户管理

#### 获取所有用户数量
- **URL**: `GET /admin/getAllUsersCount`

#### 获取用户列表
- **URL**: `POST /admin/getAllUsers`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "username": "用户",
  "email": "@example.com"
}
```

#### 新增用户
- **URL**: `POST /admin/addUser`
- **请求体**:
```json
{
  "username": "new_user",
  "password": "Pass123!",
  "email": "new@example.com"
}
```

#### 更新用户信息
- **URL**: `PUT /admin/updateUser`
- **请求体**: 同 `UserDTO`

#### 更新用户状态
- **URL**: `PATCH /admin/updateUserStatus/{id}/{status}`
- **说明**: status: 0=禁用, 1=启用

#### 删除用户
- **URL**: `DELETE /admin/deleteUser/{id}`

#### 批量删除用户
- **URL**: `DELETE /admin/deleteUsers`
- **请求体**: `[1, 2, 3]`

### 歌手管理

#### 获取歌手数量
- **URL**: `GET /admin/getAllArtistsCount?gender=1&area=内地`

#### 获取歌手列表
- **URL**: `POST /admin/getAllArtists`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "name": "周杰伦"
}
```

#### 新增歌手
- **URL**: `POST /admin/addArtist`
- **请求体**:
```json
{
  "name": "周杰伦",
  "gender": 1,
  "area": "台湾",
  "introduction": "华语流行歌手"
}
```

#### 更新歌手信息
- **URL**: `PUT /admin/updateArtist`
- **请求体**: 同上

#### 更新歌手头像
- **URL**: `PATCH /admin/updateArtistAvatar/{id}`
- **Content-Type**: `multipart/form-data`
- **参数**: `avatar` (文件)

#### 删除歌手
- **URL**: `DELETE /admin/deleteArtist/{id}`

#### 批量删除歌手
- **URL**: `DELETE /admin/deleteArtists`
- **请求体**: `[1, 2, 3]`

### 歌曲管理

#### 获取歌曲数量
- **URL**: `GET /admin/getAllSongsCount?style=流行`

#### 获取所有歌手名称
- **URL**: `GET /admin/getAllArtistNames`
- **响应**: `[{id: 1, name: "周杰伦"}, ...]`

#### 根据歌手获取歌曲
- **URL**: `POST /admin/getAllSongsByArtist`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "artistId": 1
}
```

#### 添加歌曲
- **URL**: `POST /admin/addSong`
- **请求体**:
```json
{
  "name": "稻香",
  "artistId": 1,
  "album": "魔杰座",
  "style": "流行",
  "duration": "3:42"
}
```

#### 更新歌曲信息
- **URL**: `PUT /admin/updateSong`
- **请求体**: 同上

#### 更新歌曲封面
- **URL**: `PATCH /admin/updateSongCover/{id}`
- **Content-Type**: `multipart/form-data`
- **参数**: `cover` (文件)

#### 更新歌曲音频
- **URL**: `PATCH /admin/updateSongAudio/{id}`
- **Content-Type**: `multipart/form-data`
- **参数**: `audio` (文件), `duration` (时长字符串如 "3:42")

#### 删除歌曲
- **URL**: `DELETE /admin/deleteSong/{id}`

#### 批量删除歌曲
- **URL**: `DELETE /admin/deleteSongs`
- **请求体**: `[1, 2, 3]`

### 歌单管理

#### 获取歌单数量
- **URL**: `GET /admin/getAllPlaylistsCount?style=华语`

#### 获取歌单列表
- **URL**: `POST /admin/getAllPlaylists`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10,
  "title": "华语"
}
```

#### 新增歌单
- **URL**: `POST /admin/addPlaylist`
- **请求体**:
```json
{
  "title": "我的最爱",
  "style": "流行",
  "introduction": "个人收藏"
}
```

#### 更新歌单信息
- **URL**: `PUT /admin/updatePlaylist`
- **请求体**: 同上

#### 更新歌单封面
- **URL**: `PATCH /admin/updatePlaylistCover/{id}`
- **Content-Type**: `multipart/form-data`
- **参数**: `cover` (文件)

#### 删除歌单
- **URL**: `DELETE /admin/deletePlaylist/{id}`

#### 批量删除歌单
- **URL**: `DELETE /admin/deletePlaylists`
- **请求体**: `[1, 2, 3]`

### 轮播图管理

#### 获取轮播图列表
- **URL**: `POST /admin/getAllBanners`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10
}
```

#### 添加轮播图
- **URL**: `POST /admin/addBanner`
- **Content-Type**: `multipart/form-data`
- **参数**: `banner` (文件)

#### 更新轮播图
- **URL**: `PATCH /admin/updateBanner/{id}`
- **Content-Type**: `multipart/form-data`
- **参数**: `banner` (文件)

#### 更新轮播图状态
- **URL**: `PATCH /admin/updateBannerStatus/{id}?status=1`
- **说明**: status: 0=禁用, 1=启用

#### 删除轮播图
- **URL**: `DELETE /admin/deleteBanner/{id}`

#### 批量删除轮播图
- **URL**: `DELETE /admin/deleteBanners`
- **请求体**: `[1, 2, 3]`

### 反馈管理

#### 获取反馈列表
- **URL**: `POST /admin/getAllFeedbacks`
- **请求体**:
```json
{
  "pageNum": 1,
  "pageSize": 10
}
```

#### 删除反馈
- **URL**: `DELETE /admin/deleteFeedback/{id}`

#### 批量删除反馈
- **URL**: `DELETE /admin/deleteFeedbacks`
- **请求体**: `[1, 2, 3]`

---

## 数据模型

### UserDTO
```json
{
  "username": "string"
}
```

### SongAddDTO
```json
{
  "name": "string",
  "artistId": "long",
  "album": "string",
  "style": "string",
  "duration": "string"
}
```

### SongUpdateDTO
```json
{
  "id": "long",
  "name": "string",
  "artistId": "long",
  "album": "string",
  "style": "string",
  "duration": "string"
}
```

### PlaylistAddDTO / PlaylistUpdateDTO
```json
{
  "id": "long (仅更新时需要)",
  "title": "string",
  "style": "string",
  "introduction": "string"
}
```

### ArtistAddDTO / ArtistUpdateDTO
```json
{
  "id": "long (仅更新时需要)",
  "name": "string",
  "gender": "integer (0=未知, 1=男, 2=女)",
  "area": "string",
  "introduction": "string"
}
```

---

## 错误码说明

| code | 说明 |
|------|------|
| 0 | 操作成功 |
| 1 | 操作失败 |
| 401 | 未登录 / Token无效 |
| 403 | 无权限访问 |