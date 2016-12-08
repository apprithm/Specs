require 'uri'
require 'fileutils'
require 'multi_json'
require 'net/ssh'

desc '镜像一个 github 包至 gitlab 仓库'
task :clone, [:name] do |t, p|
  name = p[:name]
  current_path = Dir.pwd

  specs = Dir[File.join(File.expand_path('~'), '.cocoapods/repos/master/Specs/*')]

  repo = specs.select { |s| File.basename(s) == name }.first

  if repo
    puts " * found repo, copy it here"
    repo_store_path = File.join(current_path, 'Specs')
    FileUtils.cp_r repo, repo_store_path

    puts " * updating repo url"
    Dir["#{repo_store_path}/#{name}/*"].each do |f|
      pod_file = File.join(f, "#{name}.podspec.json")
      json = File.read(pod_file)
      data = MultiJson.load json

      if data['source']['git']
        puts " -> #{data['version']}: git"
        orginal_repo_url = data['source']['git']
        coverted_repo_name =  URI.parse(orginal_repo_url).path[1..-1].gsub('/', '-').downcase
        data['source']['git'] = "http://tokyo.apprithm.com/mirrors/#{coverted_repo_name}"

        File.write(pod_file, JSON.pretty_generate(data))
      else data['source']['http']
        puts " -> #{data['version']}: http url, do you want speed up?"
      else data['source']['svn']
        puts " -> #{data['version']}: svn repo, do you want speed up?"
      end
    end
  else
    puts "Not find spec named: #{name}"
  end
end

desc 'gitlab 服务器镜像 Cocoapod Spec'
task :mirror, [:repo] do |t, p|
  host        = '172..0.1'
  user        = 'icyleaf'
  options     = {:keys => '~/.ssh/keys/id_rsa.pub'}

  puts "Connect gitlab server and mirror"
  Net::SSH.start(host, user, options) do |ssh|
    gitmirror_path = '/home/gitmirror/gitlab-mirrors'
    cmd = "sudo -u gitmirror -H rake \"add[#{p[:repo]}]\""
    stdout = ssh.exec!("echo 'cd #{gitmirror_path} && #{cmd}'")
    puts stdout
    ssh.loop
  end
end
