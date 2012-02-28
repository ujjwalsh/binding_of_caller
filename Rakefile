dlext = Config::CONFIG['DLEXT']
direc = File.dirname(__FILE__)

$:.unshift 'lib'

PROJECT_NAME = "binding_of_caller"

require 'rake/clean'
require 'rubygems/package_task'

require "#{PROJECT_NAME}/version"

CLOBBER.include("**/*.#{dlext}", "**/*~", "**/*#*", "**/*.log", "**/*.o")
CLEAN.include("ext/**/*.#{dlext}", "ext/**/*.log", "ext/**/*.o",
              "ext/**/*~", "ext/**/*#*", "ext/**/*.obj", "**/*#*", "**/*#*.*",
              "ext/**/*.def", "ext/**/*.pdb", "**/*_flymake*.*", "**/*_flymake")

def apply_spec_defaults(s)
  s.name = PROJECT_NAME
  s.summary = "Retrieve the binding of a method's caller. Can also retrieve bindings even further up the stack. Currently only works for MRI 1.9.2+"
  s.version = BindingOfCaller::VERSION
  s.date = Time.now.strftime '%Y-%m-%d'
  s.author = "John Mair (banisterfiend)"
  s.email = 'jrmair@gmail.com'
  s.description = s.summary
  s.require_path = 'lib'
  s.add_development_dependency("bacon","~>1.1")
  s.homepage = "http://github.com/banister/binding_of_caller"
  s.has_rdoc = 'yard'
  s.files = `git ls-files`.split("\n")
  s.test_files = `git ls-files -- test/*`.split("\n")
end

desc "Run tests"
task :test do
  sh "bacon -Itest -rubygems test.rb -q"
end

task :pry do
  puts "loading binding_of_caller into pry"
  sh "pry -r ./lib/binding_of_caller"
end

desc "generate gemspec"
task :gemspec => "ruby:gemspec"

namespace :ruby do
  spec = Gem::Specification.new do |s|
    apply_spec_defaults(s)
    s.platform = Gem::Platform::RUBY
    s.extensions = ["ext/#{PROJECT_NAME}/extconf.rb"]
  end

  Gem::PackageTask.new(spec) do |pkg|
    pkg.need_zip = false
    pkg.need_tar = false
  end

  desc  "Generate gemspec file"
  task :gemspec do
    File.open("#{spec.name}.gemspec", "w") do |f|
      f << spec.to_ruby
    end
  end
end

namespace :rbx do
  spec = Gem::Specification.new do |s|
    apply_spec_defaults(s)
    s.platform = Gem::Platform::RUBY
  end

  Gem::PackageTask.new(spec) do |pkg|
    pkg.need_zip = false
    pkg.need_tar = false
  end
end

desc "build the binaries"
task :compile do
  chdir "./ext/#{PROJECT_NAME}/" do
    sh "ruby extconf.rb"
    sh "make clean"
    sh "make"
    sh "cp *.#{dlext} ../../lib/"
  end
end

desc "reinstall gem"
task :reinstall => :gems do
  sh "gem uninstall binding_of_caller" rescue nil
  sh "gem install #{direc}/pkg/#{PROJECT_NAME}-#{BindingOfCaller::VERSION}.gem"
end

desc "build all platform gems at once"
task :gems => [:clean, :rmgems, "ruby:gem", "rbx:gem"]

task :gem => [:gems]

task :rbxgem => "rbx:gem"

desc "remove all platform gems"
task :rmgems => ["ruby:clobber_package"]

desc "build and push latest gems"
task :pushgems => :gems do
  chdir("./pkg") do
    Dir["*.gem"].each do |gemfile|
      sh "gem push #{gemfile}"
    end
  end
end
