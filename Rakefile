# -*- coding: utf-8 -*-
$:.unshift("/Library/RubyMotion/lib")
require 'motion/project/template/ios'

begin
  require 'bundler'
  Bundler.require
rescue LoadError
end

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'motion-turbolinks-demo'
  app.frameworks += ['WebKit'] # TODO: this should be moved to and applied by the gem

  app.info_plist['NSAppTransportSecurity'] = {'NSAllowsArbitraryLoads' => true}
end
