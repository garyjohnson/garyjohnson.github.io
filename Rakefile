# Mercilessly lifted from https://gist.github.com/mlafeldt/706f0ce277d7c025380b
require 'rake/clean'

DRAFTS_DIR = '_drafts'
POSTS_DIR = '_posts'
BUILD_DIR = '_site'
DEPLOY_DIR = '_deploy'
DEPLOY_BRANCH = 'master'

CLEAN.include BUILD_DIR
CLOBBER.include DEPLOY_DIR

desc 'Build the site'
task :build do
  sh 'jekyll', 'build'
end

desc 'Start web server to preview site'
task :preview do
  sh 'jekyll', 'serve', '--watch', '--drafts',
     '--port', ENV.fetch('PORT', '4000'),
     '--config', '_config.yml,_config_dev.yml'
end

desc 'Create a new draft'
task :new_draft, :title do |t, args|
  title = args[:title] || 'New Draft'
  filename = File.join(DRAFTS_DIR, "#{title.to_url}.md")

  puts "==> Creating new draft: #{filename}"
  open(filename, 'w') do |f|
    f << "---\n"
    f << "layout: post\n"
    f << "title: \"#{title.to_html(true)}\"\n"
    f << "comments: false\n"
    f << "categories:\n"
    f << "---\n"
    f << "\n"
    f << "Add awesome content here.\n"
  end
end

desc 'Create a new post'
task :new_post, :title do |t, args|
  title = args[:title] || 'New Post'
  timestamp = Time.now.strftime('%Y-%m-%d')
  filename = File.join(POSTS_DIR, "#{timestamp}-#{title.to_url}.md")

  puts "==> Creating new post: #{filename}"
  open(filename, 'w') do |f|
    f.write "---\n"
    f.write "layout: post\n"
    f.write "title: \"#{title.to_html(true)}\"\n"
    f.write "categories:\n"
    f.write "---\n"
    f.write "\n"
    f.write "Add awesome post content here.\n"
  end
end

desc 'Create a new page'
task :new_page, :title do |t, args|
  title = args[:title] || 'New Page'
  filename = File.join(title.to_url, 'index.md')

  puts "==> Creating new page: #{filename}"
  mkdir_p title.to_url
  open(filename, 'w') do |f|
    f.write "---\n"
    f.write "layout: page\n"
    f.write "title: \"#{title.to_html(true)}\"\n"
    f.write "---\n"
    f.write "\n"
    f.write "Add awesome page content here.\n"
  end
end

task :default => :build
