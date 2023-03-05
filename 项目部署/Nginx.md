## Nginx设置

### 清理和更新服务器

```shell
$ apt clean all && sudo apt update && sudo apt dist-upgrade
rm -rf /var/www/html
mkdir /var/www/app
$ vim /var/www/app/index.html
```

### 安装Nginx

```shell
$ apt install nginx
```

#### 安装和配置防火墙

```shell
$ apt install ufw
$ ufw enable
$ ufw allow "Nginx Full"
```

#### 启动Nginx并查看页面

```shell
$ systemctl start nginx
```

输入ip地址查看页面

#### 删除默认服务器配置

```shell
 rm /etc/nginx/sites-available/default
 rm /etc/nginx/sites-enabled/default
```

#### 配置

```shell
$ vim /etc/nginx/sites-available/app
server {
  listen 80;

  location / {
        root /var/www/app;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
}
ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/app
```

#### 编写index.html

```shell
$ vim /var/www/app/index.html
```

#### 检查nginx

```shell
$ systemctl status
$ nginx -t
```

#### 重新加载nginx

```shell
$ sudo service nginx reload
```



### 安装git

```shell
$ apt install git
```

```shell
$ mkdir app
$ cd app
$ git clone <your repository>
```

#### 应用程序的Nginx配置

```shell
$ vim /etc/nginx/sites-available/app
location /api {
        proxy_pass http://119.23.209.109:8800;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
  
$ systemctl reload nginx
```



### 安装NVM

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
```

### 安装Node

```shell
$ nvm install 18
```

```shell
cd api
npm install
npm config set registry http://registry.npm.taobao.org
# 查看设置是否成功
npm get registry 
npm install -g pnpm
```



### 启动项目

```shell
$ node index.js
```

#### 安装PM2

```shell
$ npm i -g pm2
```

##### 创建PM2实例

```shell
$ pm2 start --name api index.js   
$ pm2 startup ubuntu 
$ pm2 unstartup ubuntu
```

##### 停止pm2实例

```shell
$ pm2 stop api
$ pm2 stop [process_id]
```

##### 停止所有pm2实例

```shell
$ pm2 stop all
```

##### 停止多个pm2实例

```shell
$ pm2 stop app1 app3 app4
```

##### 停止和删除pm2实例 api

```shell
$ pm2 delete api
```

##### 删除pm2所有实例

```shell
$ pm2 delete all
```

##### 部署客户端项目

```shell
$ rm -rf /var/www/app/*
$ mkdir /var/www/app/client
$ cp -r dist/* /var/www/app/client
```

##### 配置项目nginx

```shell
$ vim /etc/nginx/sites-available/app
 location / {
        root /var/www/app/client/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
  $ systemctl reload nginx
```

```shell
$ vim /etc/nginx/sites-available/app

  server {
  listen 80;
  location / {
        root /var/www/app/client/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
  location /admin {
        root /var/www/story/admin/;
        proxy_pass /var/www/story/admin/
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
  location /api {
        proxy_pass http://119.23.209.109:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
   }
 }
  $ systemctl reload nginx
  
  
  
```

```shell
server {
  listen 80;
  server_name storygoo.fun www.storygoo.fun;

  location / {
        root /var/www/story/client/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }

}

server {
  listen 80;
  server_name api.storygoo.fun;
  location / {
        proxy_pass http://119.23.209.109:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }

}

server {
  listen 80;
  server_name admin.storygoo.fun;
  location / {
        root /var/www/story/admin/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }

```

```shell
 server {
  listen 80;
  location / {
        root /var/www/story/client/;
        index  index.html index.htm;
        admin 
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
   location /admin {
        proxy_pass http://119.23.209.109:8080;
   }
  location /api {
        proxy_pass http://119.23.209.109:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
   }
 }
 server {
  listen 8080;
  location / {
        root /var/www/story/admin/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
  }
```

### 开启Gzip

```shell
$ vim /etc/nginx/nginx.conf

http {

        ##
        # Gzip Settings
        ##

        gzip on;
        
        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```
