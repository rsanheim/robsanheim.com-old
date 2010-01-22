$: << File.expand_path(File.join(File.dirname(__FILE__), "lib"))

#gem 'tomafro-jekyll', '0.5.3.2'
#require 'jekyll'
require 'set'
require 'fileutils'
require 'hpricot'
require 'date'
require 'fileutils'

DESTINATION = "build"
TAGS_DIR = "site/tags"

def generate_index(posts, file, options = {})
  file.puts YAML.dump(options.merge({'layout' => 'default', 'robots' => 'noindex'}))
  file.puts "---"
  posts = posts.sort{|x, y| x.date <=> y.date }.reverse!
  output = posts.collect do |post| 
    %{{% post #{post.url} %}}
  end.join("\n")
  file.puts(output)
end

def globs(source)
  Dir.chdir(source) do
    dirs = Dir['*'].select { |x| File.directory?(x) }
    dirs -= ['_site']
    dirs = dirs.map { |x| "#{x}/**/*" }
    dirs += ['*']
  end
end

namespace :jekyll do
  task :initialize do
    gem "jekyll"
    # gem 'tomafro-jekyll', '0.5.3.6'
    require 'jekyll'
    require 'tomafro/jekyll/tags/post'
    require 'tomafro/jekyll/tags/related'
    @options = Jekyll.configuration('auto' => false, 'source' => 'site', 'destination' => DESTINATION)
    @site = Jekyll::Site.new(@options)
    @site.read_posts('')
  end
end

namespace :render do
  task :auto => ['render'] do
    require 'directory_watcher'

    puts "Auto-regenerating enabled: #{'site'} -> #{DESTINATION}"

    dw = DirectoryWatcher.new('site')
    dw.interval = 1
    dw.glob = globs('site')

    dw.add_observer do |*args|
      t = Time.now.strftime("%Y-%m-%d %H:%M:%S")
      puts "[#{t}] regeneration: #{args.size} files changed"
      Rake::Task['build:all'].invoke
      @site.process
    end

    dw.start
    
    loop { sleep 500 }
  end
end

task :render => 'jekyll:initialize' do 
  FileUtils.rm_rf(DESTINATION)
  @site.process
end

namespace :build do
  task :all => ['build:tags', 'build:archive:by_month', 'build:archive:by_year']
  
  task :tags => 'jekyll:initialize' do
    FileUtils.rm_rf(TAGS_DIR)
    FileUtils.mkdir_p(TAGS_DIR)
    tags = @site.categories
    tags.each do |tag, posts|
      File.open(File.join(TAGS_DIR, tag + ".html"), "w") do |file|
        generate_index posts, file, 'title' => "Posts tagged with #{tag}"
      end
    end
  end
  
  namespace :archive do
    task :by_month => 'jekyll:initialize' do
      by_year_and_month = @site.posts.inject({}) do |result, post|
        (result[[post.date.year, post.date.month]] ||= []) << post
        result
      end
      
      by_year_and_month.each do |(year, month), posts|
        name = Date.new(2008, month, 1).strftime("%B")
        path = File.join("site", year.to_s, "%02i.html" % month)
        FileUtils.mkdir_p(File.dirname(path))
        File.open(path, "w") do |file|
          generate_index posts, file, 'title' => "Posts from #{name} #{year}"
        end
      end
    end
    
    task :by_year => 'jekyll:initialize' do
      by_year = @site.posts.inject({}) do |result, post|
        (result[post.date.year] ||= []) << post
        result
      end

      by_year.each do |year, posts|
        path = File.join("site", year.to_s + ".html")
        FileUtils.mkdir_p(File.dirname(path))
        File.open(path, "w") do |file|
          generate_index posts, file, 'title' => "Posts from #{year}"
        end
      end
    end
  end
end