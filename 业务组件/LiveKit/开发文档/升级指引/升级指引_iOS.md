# iOS 升级指引

iOS 升级互动直播组件版本只需修改项目 'Podfile' 脚本文件中依赖的版本号即可：

```ruby
target 'App' do
  ## 指定要依赖的互动直播组件版本
  pod 'NELiveKit', '~> 2.0.0'
end
```

