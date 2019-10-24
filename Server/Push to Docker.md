# Push local image to hub.docker.com

**Clean unused images**
```shell
docker image prune
```

**Build image locally**

```yml
    #    container_name: infoma-api
    build: ./api
```

```shell
docker-compose up --build api
```

```shell
> docker login

> docker images|grep infoma
infoma_api                       latest              efc4f078437f        12 minutes ago      1.27GB
infoma_loc-api                   latest              c574dde0da89        8 months ago        1.04GB
> docker tag efc4f078437f hustmck/infoma_api:enablecaching
> docker push hustmck/infoma_api
The push refers to repository [docker.io/hustmck/infoma_api]
4edb48a50bb4: Pushed
2e635e468657: Pushed
2cb0010274d0: Pushed
097b6a43edd8: Pushed
2c9e06ddd139: Pushed
ffab8273c674: Mounted from library/python
e6f6cbe5e14e: Mounted from library/python
3eb255f8a6ac: Mounted from library/python
b860b6c48eec: Mounted from library/python
1fa8778eb779: Mounted from library/python
fa0c3f992cbd: Mounted from library/python
ce6466f43b11: Mounted from library/python
719d45669b35: Mounted from library/python
3b10514a95be: Mounted from stilliard/pure-ftpd
enablecaching: digest: sha256:0c9ecad35a3d30f950e6c98d72cda3ce546ac24a77504681359eceb4524464cf size: 3267
```

![](media/Screen%20Shot%202019-09-28%20at%204.43.04%20PM.png)


**Pull and Apply new image**
Edit docker-compose.yml
```yml
  api:
    # build: ./api
    image: hustmck/infoma_api:enablecaching
    container_name: infoma-api
```

**Update Image**
```shell
docker pull hustmck/infoma_api:enablecaching
```