---
layout: post
title: Oauth2 on Rails
category: Ruby
date: 2016-09-17
tag: 
- 记录
---

> OAuth2是一个关于授权（authorization）的开放网络标准。

<!-- more -->

### Oauth身份验证过程身份认证的服务平台,和一个客服端程序做为用户认证。Rails通常使用devise, doorkeeper and omniauth-oauth2 gems构建Oauth应用。

## The server application

### Devise gem将会提供身份验证, doorkeeper gem 将允许应用做为一个Oauth2 server。

### 添加 devise and doorkeeper gems 到Gemfile

```ruby
gem "devise"
gem "doorkeeper"
``` 

### 生成这gem的引用和迁移文件。

```ruby
rails g devise:install
rails g devise User

rails g doorkeeper:install
rails generate doorkeeper:migration

rake db:migrate
``` 

###  这过程会有很多文件生成,配置doorkeeper.rb文件,修改doorkeeper的初始化，让它使用devise作为认证。

```ruby
resource_owner_authenticator do
  current_user || begin
    session[:user_return_to] = request.fullpath
    redirect_to new_user_session_url
  end
end

admin_authenticator do
  current_user || redirect_to(new_user_session_url)
end
``` 

### The resource_owner_authenticator这代码块的意思:允许 devise 认证的用户可以访问doorkeeper的资源。假如用户没有通过认证,就存储url,跳转到登录页面。The admin_authenticator bock 做为访问用户限制。它用来管理客服端应用程序的所需令牌身份验证的限制。

### 创建控制器，为客服端提供认证用户的信息。

```ruby
class ApplicationController < ActionController::Base
  before_action :doorkeeper_authorize!, only: :me

  def me
    render json: User.find(doorkeeper_token.resource_owner_id).as_json
  end
end
```

### doorkeeper_authorize! 检测请求oauth的授权码是否正确,如果错误将渲染401 Unauthorized的请求头。

### 不能忘记更新路由

```ruby
Rails.application.routes.draw do
  use_doorkeeper
  devise_for :users

  get '/me' => 'application#me'
end
```

### 客服端应用需要被注册，在 server application平台上。访问http://localhost:3000/oauth/applications/new，注册新应用。你将会跳转到一个认证页面，按照规定要求填写信息后，将会被返回client_id,secret信息。