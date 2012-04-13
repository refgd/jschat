# This deploys by using gem and symlinking jschat to a path suitable for a web server

deploy_to = 'web2'
app_path = '/var/www/jschat'
remote_gem_command = 'gem'
restart_web_server = '/etc/init.d/apache2 restart'

task :test do
  require 'rake/testtask'
  Rake::TestTask.new do |t|
    t.libs << 'test'
    t.test_files = FileList['test/*_test.rb']
    t.verbose = true
  end
end

task :assets do
  js = ''
  File.unlink('lib/jschat/http/public/javascripts/all.js')

  Dir['lib/jschat/http/public/javascripts/**/*.js'].reverse.each do |file|
    js << File.read(file)
  end

  File.open('lib/jschat/http/public/javascripts/all.js', 'w+') do |file|
    file.write js
  end
end

task :deploy do
  `#{remote_gem_command} build jschat.gemspec`
  jschat_gem = `ls jschat-*.gem`
  `#{remote_gem_command} push jschat-*.gem`
  puts "Waiting until gem has updated on rubyforge"
  sleep 30
  `ssh #{deploy_to} sudo #{remote_gem_command} update jschat`
  gem_path = `ssh #{deploy_to} #{remote_gem_command} environment | grep "  - INSTALLATION DIRECTORY: " | sed 's/  - INSTALLATION DIRECTORY: //'`
  `ssh #{deploy_to} rm #{app_path}`
  jschat_gem_path = jschat_gem.sub(/\.gem/, '').strip
  `ssh #{deploy_to} ln -s #{gem_path.strip}/#{jschat_gem_path}/lib/jschat/http/ #{app_path}`
  `ssh #{deploy_to} sudo #{restart_web_server}`
  `rm jschat-*.gem`
end
