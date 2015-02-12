def run_command(command)
  if File.exist?('Gemfile.lock')
    sh %(bundle exec #{command})
  else
    sh %(chef exec #{command})
  end
end

task :destroy_all do
  Rake::Task[:destroy_machines].invoke
  run_command('rm -rf Gemfile.lock && rm -rf Berksfile.lock && rm -rf cookbooks/')
end

desc "Destroy machines"
task :destroy_machines do
  run_command('chef-client -z destroy_all.rb')
end

desc "Vendor your cookbooks/"
task :berks_vendor do
  run_command('berks vendor cookbooks')
end

desc "All-in-One Neutron build"
task :aio_neutron do
  run_command('chef-client -z vagrant_linux.rb aio-neutron.rb')
end

desc "All-in-One Nova-networking build"
task :aio_nova do
  run_command('chef-client -z vagrant_linux.rb aio-nova.rb')
end

desc "Multi-Neutron build"
task :multi_neutron do
  run_command('chef-client -z vagrant_linux.rb multi-neutron.rb')
end

desc "Multi-Nova-networking build"
task :multi_nova do
  run_command('chef-client -z vagrant_linux.rb multi-nova.rb')
end

desc "Blow everything away"
task clean: [:destroy_all]
