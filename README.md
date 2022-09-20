# ledocker
a docker enviroment for phper

### 包含组件
mongo
mysql
nginx
php-fpm
rabbitmq
redis

### 配置文件
.env

### 拉取仓库
```
git pull https://github.com/zhangatle/ledocker.git
```

### 构建镜像
```
cd ledocker
docker-compose build php-fpm nginx mysql redis mongo rabbitmq
```

### 启动容器
```
docker-compose up -d php-fpm nginx mysql redis mongo rabbitmq
```
