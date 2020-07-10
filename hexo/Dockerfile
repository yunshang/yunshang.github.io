FROM mhart/alpine-node:12

WORKDIR /hexo

RUN \
  apk add --no-cache git openssh-client autoconf \ 
  automake \ 
  bash g++ libc6-compat libjpeg-turbo-dev libpng-dev \
  make nasm \
  && npm config set unsafe-perm true \
  && npm install hexo-cli -g \
  && npm install yarn -g \
  && npm install gifsicle -g \
  && npm install pngquant-bin -g \
  && hexo init . \
  && yarn install \
  && yarn add hexo-deployer-git \
  && yarn add  hexo-wordcount \
  && yarn add hexo-all-minifier \
  && yarn add hexo-generator-feed \
  && yarn add hexo-admin
  

VOLUME ["/hexo/source", "/hexo/themes", "/root/.ssh"]

EXPOSE 4000

COPY hexo-deploy.sh /hexo/hexo-deploy.sh
COPY docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ['/bin/bash']
