#task default: ["test"]

desc "Cleanup all nodes"
task :clean do
  _setup
  sh %{chef exec chef-client -z destroy_all.rb}
end

desc "All-in-One Neutron build"
task :aio_neutron do
  _setup
  sh %{chef exec chef-client -z vagrant_linux.rb aio-neutron.rb}
end

desc "All-in-One Nova-networking build"
task :aio_nova do
  _setup
  sh %{chef exec chef-client -z vagrant_linux.rb aio-nova.rb}
end

desc "Multi-Neutron build"
task :multi_neutron do
  _setup
  sh %{chef exec chef-client -z vagrant_linux.rb multi-neutron.rb}
end

desc "Multi-Nova-networking build"
task :multi_nova do
  _setup
  sh %{chef exec chef-client -z vagrant_linux.rb multi-nova.rb}
end

# Setup the chef client logging level according to the Rake output level
def _setup
  ENV['CHEF_DRIVER'] = 'vagrant'
end
