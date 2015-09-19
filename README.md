# ChinaCity 省市区级联选择

[![Build Status](https://travis-ci.org/saberma/china_city.png?branch=master)](https://travis-ci.org/saberma/china_city)

![china_city](http://cl.ly/image/3c212i1e3b1T/ScreenFlow.mp4.gif)

支持 Rails3.1, Rails3.2, Rails4.0。

请留意，Rails3.1 与 Ruby2.0 不兼容，sprockets 无法正常解析 application.js，请使用 Ruby1.9。

## 简介

这是一个基于 Rails Engine 开发的插件，为 Rails 项目增加省市区三级（或者省市 二级）选择框，可用于实现收货地址等信息的录入。

## 安装

### Gemfile

    gem 'china_city', git: 'https://github.com:cheenwe/china_city.git'

### app/assets/javascripts/application.js

    //= require 'jquery'
    //= require 'china_city/jquery.china_city'

### config/routes.rb

    mount ChinaCity::Engine => '/china_city'

## 使用

框架：rails + AngularJS

在form页面 代码 格式

```ruby
  .form-group.col-md-12
    = f.label :city, class: "label-title col-md-1"
    = ng_city f
```

ng_city 方法

```ruby
  def ng_city(f)
    htmls = []
    {provinces: :cities, cities: :areas, areas: nil}.each do |parents, subs|
      parent = parents.to_s.singularize
      label_and_select = []
      ng_change = subs ? "city_option_change(current.#{parent}, '#{subs}')" : "city_option_change(current.#{parent}, null)"
        html = content_tag :select, class: "city-select form-control col-xs-1", "ng-options" => "option[1] as option[0] for option in city_options.#{parents}", "ng-model" => "current.#{parent}", "ng-change"=>ng_change do
          content_tag :option, t("helpers.options.#{parents}"), value: ""
        end
      label_and_select << html.html_safe
      html = content_tag :div, label_and_select.join("").html_safe, class: "col-md-2"
      htmls << html.html_safe
    end
    htmls.join("").html_safe
  end

```


AngularJS 获取数据

```ruby
  $scope.city_options = {provinces: <%= ChinaCity.list.to_s.html_safe %>};
  $scope.city_option_change = function(code, sub_collection){
    if(sub_collection){
      var url = "/china_city/" + code;
      $http.get(url).success(function(resp){
        $scope.city_options[sub_collection] = resp;
      });
    }
  }

  $scope.after_show = function(resp){
    if($scope.city_options){
      if($scope.current.province){
        $scope.city_option_change($scope.current.province, 'cities');
      }

      if($scope.current.city){
        $scope.city_option_change($scope.current.city, 'areas');
      }
    }
  }
```




请留意：所有选择框都要有 `city-select` class，并都包含于 class='city-group' 的 DOM 元素之下。

选择后的值为国家地区编码，如深圳市的为 `440300`，可通过调用 `ChinaCity.get('440300')` 将编码转化为城市名称。

## 贡献

```bash
git clone git@github.com:saberma/china_city.git
cd china_city
rake appraisal:install
cd spec/dummy
rails server # http://localhost:3000/china_city
```

## 测试

```bash
rvm use 2.0.0
rake appraisal:rails4 spec
rake appraisal:rails32 spec
rvm use 1.9.3
bundle install
rake appraisal:rails31 spec
```

## 类似项目

* https://github.com/Kehao/area_cn_select
