---
layout: post
title: 一个客服端应用
category: Ruby
date: 2016-09-17
tag: 
- 思考
---
> 第一篇我们构建了Oauth 服务应用,现在我们创建一个发客服端应用。

<!-- more -->

###  omniauth-oauth2 gem,允许我们在试用Oauth2的时候，仅通过一个简单的配置文件(strategy)就可以办到。

```ruby
gem 'omniauth-oauth2'
``` 

### Omniauth 是一个通用gem,它允许程序员用它使用各种oath提供者，通过不同的配置。

### 在lib下创建doorkeeper.rb

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    class Doorkeeper < OmniAuth::Strategies::OAuth2
      option :name, 'doorkeeper'
      option :client_options, {
        site:          'http://localhost:3000',
        authorize_url: 'http://localhost:3000/oauth/authorize'
      }

      uid {
        raw_info['id']
      }

      info do
        {
          email: raw_info['email'],
        }
      end

      extra do
        { raw_info: raw_info }
      end

      def raw_info
        @raw_info ||= access_token.get('/me').parsed
      end
    end
  end
end
``` 

###  再生成omniauth的配置文件

```ruby
touch config/initializers/omniauth.rb
``` 

```ruby
require 'doorkeeper'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :doorkeeper, <application_id>, <application_secret>
end
``` 

### application_id 和 application_secret 填入上一篇文章中得到的值。

### 配置路由

```ruby
Rails.application.routes.draw do
  root to: redirect('/auth/doorkeeper')

    get '/auth/:provider/callback' => 'application#authentication_callback'
    end
  end
``` 

### 第一行doorkeeper的认证路由，将会被omniauth处理，第二行服务器的回调地址，将会有身份验证。

```ruby
class ApplicationController < ActionController::Base
  auth = request.env['omniauth.auth']
    render json: auth.to_json
    end
  end
``` 

### 参考资料
- http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html

-  https://ruby-china.org/topics/14656

-  http://linjunzhu.github.io/blog/2014/09/21/doorkeepershi-yong-jian-jie/
