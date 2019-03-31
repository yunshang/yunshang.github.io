---
layout: post
title: 【译】Rails 集成第三方 API
category: 翻译
date: 2017-10-23
tag: 
- 翻译
---

> 在搭建自己网站的时候，总会遇到集成第三方应用的时候，例如你不想自己收集数据，或者是要用到别人的服务。这文章将向您展示一个快速和简单的例子,如何使用API来填充应用程序用一些有用的数据。

<!-- more -->

![](http://7te7uy.com1.z0.glb.clouddn.com/1-oVHo2zmRuVRN7BBp4HTD_A.jpeg)

## 构建 API 基础

我非常喜欢做菜，所以决定通过做一个菜谱 app。我将使用通过 Mashape 提供的 Spoonacular API。这是 API 文档 [链接](https://market.mashape.com/spoonacular/recipe-food-nutrition), 构建 API 之前，我们需要申请一个 mashape 帐号，通过帐号生成 API token。

我们先创建一个 Rails 项目，添加我们接下来需要到的 GEM，[dotenv-rails](https://github.com/bkeepers/dotenv),能让我保存一些重要数据保存成环境变量，[faraday](https://github.com/lostisland/faraday), 简单,灵活的 HTTP 客户端。

```ruby
 gem 'dotenv-rails'
 gem 'faraday'
```

新建两个类 Connection，Request，在 lib/spoonacular 文件夹下，这两个类是与第三方 API 建立连接，和请求用的。

```ruby
require 'faraday'
require 'json'

class Connection
  BASE = 'https://spoonacular-recipe-food-nutrition-v1.p.mashape.com'

  def self.api
    Faraday.new(url: BASE) do |faraday|
      faraday.response :logger
      faraday.adapter Faraday.default_adapter
      faraday.headers['Content-Type'] = 'application/json'
      faraday.headers['X-Mashape-Key'] = ENV['MASHAPE_KEY']
    end
  end
end
```

在 Connection 类里，定义了连接 API 的方法，和一些必须的配置文件。

```ruby
class Request
  class << self
    def where(resource_path, query = {}, options = {})
      response, status = get_json(resource_path, query)
      status == 200 ? response : errors(response)
    end

    def get(id)
      response, status = get_json(id)
      status == 200 ? response : errors(response)
    end

    def errors(response)
      error = { errors: { status: response["status"], message: response["message"] } }
      response.merge(error)
    end

    def get_json(root_path, query = {})
      query_string = query.map{|k,v| "#{k}=#{v}"}.join("&")
      path = query.empty?? root_path : "#{root_path}?#{query_string}"
      response = api.get(path)
      [JSON.parse(response.body), response.status]
    end

    def api
      Connection.api
    end
  end
end
```

Request 类负责 Spoonacular API 的实际请求，还定义了一些辅佐方法。

## 数据处理

现在我们需要向 Spoonacular API 发出请求, 通过它的文档，我们需要设计自己数据结构。这个 APP 只是想简单罗列出食谱，食谱有很多成分，和说明。
对于初学者，我们先创建一个 Spoonacular::Base 类，其中定义了错误属性和 initializaton method, 我建议把所有类放在 app/services/spoonacular 下。

```ruby
module Spoonacular
  class Base
    attr_accessor :errors

    def initialize(args = {})
      args.each do |name, value|
        attr_name = name.to_s.underscore
        send("#{attr_name}=", value) if respond_to?("#{attr_name}=")
      end
    end
  end
end
```
我们继续完善 Spoonacular API， 我们需要定义两条 routes, 一条是菜谱列表，另一个是菜谱详细信息。

```ruby
GET recipes/random
GET recipes/:id/information
```

正如你期望的，我们再创建 Spoonacular::Recipe 类，让它继承 Spoonacular::Base, 这个类有随机查找菜谱的的方法，我们还需要重新定义初始方法，以解析 HTTP 响应中的结构。

```ruby
module Spoonacular
  class Recipe < Base
    attr_accessor :aggregate_likes,
                  :dairy_free,
                  :gluten_free,
                  :id,
                  :image,
                  :ingredients,
                  :instructions,
                  :ready_in_minutes,
                  :title,
                  :vegan,
                  :vegetarian

    MAX_LIMIT = 12

    def self.random(query = {})
      response = Request.where('recipes/random', query.merge({ number: MAX_LIMIT }))
      recipes = response.fetch('recipes', []).map { |recipe| Recipe.new(recipe) }
      [ recipes, response[:errors] ]
    end

    def self.find(id)
      response = Request.get("recipes/#{id}/information")
      Recipe.new(response)
    end

    def initialize(args = {})
      super(args)
      self.ingredients = parse_ingredients(args)
      self.instructions = parse_instructions(args)
    end

    def parse_ingredients(args = {})
      args.fetch("extendedIngredients", []).map { |ingredient| Ingredient.new(ingredient) }
    end

    def parse_instructions(args = {})
      instructions = args["analyzedInstructions"]
      if instructions
        steps = instructions.first.fetch("steps", [])
        steps.map { |instruction| Instruction.new(instruction) }
      end
    end
  end
end
```

返回来响应体中，包含了每个菜谱配方，都会创建一个 Spoonacular::Recipe 对象， 和几个 Spoonacular::Lngredient, Spoonacular::Instruction 对象。

```ruby
module Spoonacular
  class Ingredient < Base
    attr_accessor :id,
                  :image,
                  :name,
                  :amount,
                  :unit,
                  :original_string
  end
end
```

```ruby
module Spoonacular
  class Instruction < Base
    attr_accessor :number, :step
  end
end
```

## 最终构建应用

我们搭建 WEB 界面。 如前所述，此界面将具有菜谱配方列表和菜谱配方详细信息。 因此，应该不难猜到我们需要为RecipesController 创建 index 和 show 页面：

```ruby
class RecipesController < ApplicationController

  def index
    @tag = query.fetch(:tags, 'all')
    @recipes, @errors = Spoonacular::Recipe.random(query)
  end

  def show
    @recipe = Spoonacular::Recipe.find(params[:id])
  end

  private
  def query
    params.fetch(:query, {})
  end
end
```

### 原文

- [https://revs.runtime-revolution.com/integrating-a-third-party-api-with-rails-5-134f960ddbba](https://revs.runtime-revolution.com/integrating-a-third-party-api-with-rails-5-134f960ddbba)

