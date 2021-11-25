---
title: 为什么 Docker Pull 显示的 Digest 和 Docker Hub 的不一样
date: 2021-11-25 11:48:29
tags:
---

这个问题困扰自己很久了，今天终于搞明白了。

复现： 在 Docker Hub 查看 alpine 的镜像信息，并在主机上执行 `docker pull alpine:3.12`
问题： 在网站上看到的 `Digest` 和控制台里显示的 `Digest` 不一样

控制台显示的 `Digest` 是 `d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280`

```bash
3.12: Pulling from library/alpine
Digest: sha256:d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280
Status: Image is up to date for alpine:3.12
docker.io/library/alpine:3.12
```

这个 `Digest` 并不在 alpine:3.12 官方仓库对应不同平台的[镜像列表](
https://hub.docker.com/_/alpine?tab=tags&page=1&name=3.12) 里

| DIGEST         | OS/ARCH      | COMPRESSED SIZE
|----------------|--------------|-----------------
|7893969ec350    |linux/386     | 2.67 MB
|[74e5ad84a67e](https://hub.docker.com/layers/alpine/library/alpine/3.12/images/sha256-74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18) |linux/amd64   | 2.68 MB
|8a6e4b2093ee    |linux/arm/v6  | 2.49 MB
|45a18fc6f681    |linux/arm/v7  | 2.31 MB
|29ba524fa7f5    |linux/arm64/v8| 2.59 MB
|08340ab4a605    |linux/ppc64le | 2.69 MB
|9a7276e7579f    |linux/s390x   | 2.46 MB

那么这两个 `Digest`，到底以那个为准呢

- d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280
- 74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18

试着都拉取一下，发现都可以

```
docker pull alpine@sha256:d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280
docker pull alpine@sha256:74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18
```

它们对应的镜像都是 `alpine:3.12` 可以说是一样但又不完全一样。这点可以通过 `docker image inspect alpine:3.12` 验证。

```json
[
    {
        "Id": "sha256:b0925e0819214cd29937af66dbaf0e6fe239997faea60922cc890f9984512507",
        "RepoTags": [
            "alpine:3.12"
        ],
        "RepoDigests": [
            "alpine@sha256:74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18",
            "alpine@sha256:d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280"
        ],
        "Created": "2021-11-12T17:20:08.442217528Z",
        "Container": "385e1cc96cc7482dfb6847e293bb24baecd3f48a49791b9b45e297204b160287",
        "...": {} 
   }
]
```

也就是两个 `Digest` 都对应着 Docker 官方打包机在 `2021-11-12T17:20:08.442217528Z` 这个时间点打包出来的镜像的 ID

```
构建链接（会被定期删除）
https://doi-janky.infosiftr.net/job/multiarch/job/amd64/job/alpine/lastSuccessfulBuild/artifact/build-info/image-ids/alpine_3.12.txt
镜像 ID
sha256:b0925e0819214cd29937af66dbaf0e6fe239997faea60922cc890f9984512507
```

那么不同的 `Digest` 是怎么对应同一个镜像的呢，这就要查看 Docker Registry 的 API 文档并了解 Docker 拉取镜像的原理了。

## 结论

`docker pull` 显示的是跨平台的 manifest list 的 `Digest`，而 Docker Hub 显示的是各个平台 manifest 的 `Digest`。

| 请求对象        | MIME    |
|---------------|---------|
| manifest list | application/vnd.docker.distribution.manifest.list.v2+json |
| manifest      | application/vnd.docker.distribution.manifest.v2+json      |

可以通过 curl 指定不同的 `Accpet` HTTP 请求头参数测试

```bash
curl https://registry.mirror.aliyuncs.com/v2/library/alpine/manifests/3.12 \
  -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json" -v
  
curl https://registry.mirror.aliyuncs.com/v2/library/alpine/manifests/3.12 \
  -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -v
```

### Manifest List

Response Header

```yaml
Content-Type: application/vnd.docker.distribution.manifest.list.v2+json
Content-Length: 1638
Docker-Content-Digest: sha256:d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:d9459083f962de6bd980ae6a05be2a4cf670df6a1d898157bceb420342bec280"
```

Response Body

```json
{
  "manifests": [
    {
      "digest": "sha256:74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "size": 528
    },
    {
      "digest": "sha256:8a6e4b2093eee6246fdd5181cfcad4b587db1068c93315ccf5366f79d8117485",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v6"
      },
      "size": 528
    },
    "..."
  ],
  "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
  "schemaVersion": 2
}
```

### Manifest

Response Header

```yaml
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Content-Length: 528
Docker-Content-Digest: sha256:74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:74e5ad84a67e9b8ed7609b32dc2460b76175020923d7f494a73a851446222d18"
```

Response Body

```json
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1471,
      "digest": "sha256:b0925e0819214cd29937af66dbaf0e6fe239997faea60922cc890f9984512507"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 2809473,
         "digest": "sha256:8572bc8fb8a32061648dd183b2c0451c82be1bd053a4ea8fae991436b92faebb"
      }
   ]
}
```