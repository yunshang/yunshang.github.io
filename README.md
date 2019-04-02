# docker-hexo
Hexo on Docker

## Up and Down
```bash
$ docker-compose up -d
$ docker-compose down
```

## Hexo Web Preview
[http://localhost:4000](http://localhost:4000)


## Hexo Web Admin
[http://localhost:4000/admin](http://localhost:4000/admin)  



## Hexo Commands
```bash
$ docker exec hexo hexo new post "Test Post"
$ docker exec hexo hexo new page "Test Page"
$ docker exec hexo hexo generate
$ docker exec hexo hexo deploy
$ docker exec hexo new page tags
```
