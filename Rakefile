require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    message = "Site updated at #{Time.now.utc}"
    system "git add ."
    system "git commit -am #{message.shellescape}"
    system "git push origin main --force"  
    system "mv _site/* #{tmp}"
    system "git checkout -B gh-pages"
    system "rm -rf *"
    system "mv #{tmp}/* ."
    system "git add ."
    system "git commit -am #{message.shellescape}"
    system "git push origin gh-pages --force"
    system "rm -rf _site"
    system "git checkout main"
    system "echo yolo"
  end
end

task :default => :publish
