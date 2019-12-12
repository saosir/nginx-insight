- [location 匹配方式](#location-匹配方式)
  - [前缀匹配](#前缀匹配)
  - [精确匹配 `=`](#精确匹配-)
  - [正则匹配 `~`](#正则匹配-)
  - [示例](#示例)
  - [location 前缀匹配中的 slash](#location-前缀匹配中的-slash)

# location 匹配方式

基本语法

```
Syntax: location [= | ~ | ~* | ^~] uri { ... }
location @name {...}
Default: —
Context: server, location
```

## 前缀匹配

遵循最长匹配规则，假设一个请求匹配到了两个普通规则，则选择匹配长度最大的

```
location /{
}

location /test{
}

location ^~ /images {
}
```

如果匹配 `^~ /images` 不再进行正则匹配

## 精确匹配 `=`

精确匹配之后停止匹配后面 location

```
location = /{
}
```

## 正则匹配 `~`

`~` 区分大小写的匹配:

```
location ~ ^*.php${
}
```

`~*` 不区分大小写的匹配:

```
  location ~* ^*.php${
  }
```

## 示例

```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

- `/` 精确匹配 A
- `/index.html` 最长匹配 B（正则搜索未发现匹配）
- `/documents/document.html` 最长匹配 C（正则搜索未发现匹配）
- `/images/1.gif` 最长匹配 D（由于存在`^~`不会进行正则匹配，因此不会进到 E）
- `/documents/1.jpg` 正则匹配 E（最长匹配 C，但是C没有`^~`，搜索发现匹配正则 E，因此不会进到 C）

## location 前缀匹配中的 slash

如果location为前缀匹配，url以`/`结尾，并且请求会被 `proxy_pass`, `fastcgi_pass`, `uwsgi_pass`, `scgi_pass`, `memcached_pass`或者`grpc_pass`中的一个处理，如果请求结尾不带`/`，nginx会返回一个 301 重定向，如：

```
location /test/ {
    proxy_pass http://localhost:8080
}
```

当请求为 `/test` 时候，nginx返回 301 重定向到 `/test/`

解决办法：

```
location /test/ {
    proxy_pass http://user.example.com;
}

location = /test {
    proxy_pass http://login.example.com;
}
```