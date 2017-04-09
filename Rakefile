# encoding: utf-8
require 'rake/clean'
require 'rdoc'
require 'rdoc/encoding'
require 'rdoc/markup/to_html'
require 'rdoc/options'
require 'uri'
require 'nokogiri'
require 'kramdown'


def cleanup(html)
  html_doc = Nokogiri::HTML(html)
  html_doc.xpath("//h1").first.remove # Removes Sinatra Heading
  html_doc.xpath("//ul").first.remove # Removes the ToC in Markdown
  toc_header = html_doc.xpath("//h2").first
  toc_header.remove if toc_header.inner_html == "Table of Contents"
  html_doc.to_html
end


def readme(pattern = "%s", &block)
  return readme(pattern).each(&block) if block_given?
  %w[en de es fr hu ja zh ru ko pt-br pt-pt].map do |lang|
    pattern % "README#{lang == "en" ? "" : ".#{lang}"}"
  end
end

def contrib(pattern = "%s", &block)
  return contrib(pattern).each(&block) if block_given?

  %w[
    sinatra-config-file
    sinatra-multi-route
    sinatra-content-for
    sinatra-namespace
    sinatra-cookies
    sinatra-reloader
    sinatra-respond-with
    sinatra-extension
    sinatra-streaming
    sinatra-json
    sinatra-link-header
  ].map do |extension|
    pattern % extension
  end
end

def rack_protection(pattern = "%s", &block)
  return rack_protection(pattern).each(&block) if block_given?

  %w[
    rack-protection-authenticity-token
    rack-protection-content-security-policy
    rack-protection-cookie-tossing
    rack-protection-escaped-params
    rack-protection-form-token
    rack-protection-frame-options
    rack-protection-http-origin
    rack-protection-ip-spoofing
    rack-protection-json-csrf
    rack-protection-path-traversal
    rack-protection-remote-referrer
    rack-protection-remote-token
    rack-protection-session-hijacking
    rack-protection-strict-transport
    rack-protection-xss-header
  ].map do |extension|
    pattern % extension
  end
end

# generates Table of Contents
def with_toc(src)
  toc = "<div class='toc'>\n"
  last_level = 1
  src = src.gsub(/<h(\d)>(.*)<\/h\d>/) do |line|
    level, heading = $1.to_i, $2.strip
    aname = URI.escape(heading)
    toc << "\t"*last_level << "<ol class='level-#{level-1}'>\n" if level > last_level
    toc << "\t"*level << "</ol>"*(last_level - level) << "\n" if level < last_level
    toc << "\t"*level << "<li><a href='##{aname}'>#{heading}</a></li>\n"
    last_level = level
    "<a name='#{aname}'></a>\n" << line
  end
  toc << "\t" << "</ol>"*(last_level-1) << "\n</div>\n"
  toc + src
end

task :default => ['_sinatra', :build]

desc "Build outdated static files and API docs"
task :build => [:pull, 'build:static']

desc "Build outdated static files"
task 'build:static' => readme("_includes/%s.html") + contrib("_includes/%s.html") + rack_protection("_includes/%s.html")

desc "Build anything that's outdated and stage changes for next commit"
task :regen => [:build] do
  sh 'git add api _includes'
  puts "\nPrebuilt files regenerated and staged in your index. Commit with:"
  puts "  git commit -m 'Regen prebuilt files'"
end

desc 'Pull in the latest from the sinatra, sinatra-contrib and rack-protection code'
task :pull => ['pull:sinatra']

directory "_sinatra" do
  puts 'Cloning sinatra repo'
  sh "git clone git://github.com/sinatra/sinatra.git _sinatra" 
end

desc 'Pull in the latest from the sinatra repo'
task 'pull:sinatra' => "_sinatra" do
    puts 'Pulling sinatra.git'
    sh "cd _sinatra && git pull &>/dev/null"
end

readme("_sinatra/%s.md") { |fn| file fn => '_sinatra' }
file 'AUTHORS' => '_sinatra'

readme do |fn|
  file "_includes/#{fn}.html" => ["_sinatra/#{fn}.md", "Rakefile"] do |f|
    markdown_string = File.read("_sinatra/#{fn}.md").encode('UTF-16le', :invalid => :replace, :replace => "").encode("UTF-8")
    markdown_string.gsub!(/```(\s?(\w+\n))?/) do |match|
      match =~ /```\s?\n/ ? "~~~~~\n" : match.sub(/```\s?/, "~~~~")
    end
    markdown = Kramdown::Document.new(markdown_string, :fenced_code_blocks => true, :coderay_line_numbers => nil, :auto_ids => false)
    html = cleanup(markdown.to_html)
    File.open(f.name, 'w') { |io| io.write with_toc(html) }
  end
end


desc 'Build contrib docs'
task 'build:contrib_docs' do
  puts 'Building sinatra-contrib docs'
  sh "cd _sinatra/sinatra-contrib && rake doc"
end


contrib("_sinatra/sinatra-contrib/doc/%s.rdoc") { |fn| file fn => '_sinatra/sinatra-contrib' }

contrib do |fn|
  file "_includes/#{fn}.html" => ["build:contrib_docs", "_sinatra/sinatra-contrib/doc/#{fn}.rdoc", "Rakefile"] do |f|
    html =
      RDoc::Markup::ToHtml.new
      .convert(File.read("_sinatra/sinatra-contrib/doc/#{fn}.rdoc"))
    File.open(f.name, 'wb') { |io| io.write html }
  end
end

desc 'Build rack protection docs'
task 'build:rack_protection_docs' do
  puts 'Building rack-protection docs'
  sh "cd _sinatra/rack-protection && rake doc"
end


rack_protection("_sinatra/rack-protection/doc/%s.rdoc") { |fn| file fn => '_sinatra/rack-protection' }

rack_protection do |fn|
  file "_includes/#{fn}.html" => ["build:rack_protection_docs", "_sinatra/rack-protection/doc/#{fn}.rdoc", "Rakefile"] do |f|
    html =
      RDoc::Markup::ToHtml.new
      .convert(File.read("_sinatra/rack-protection/doc/#{fn}.rdoc"))
    File.open(f.name, 'wb') { |io| io.write html }
  end
end

desc 'Rebuild site under _site with Jekyll'
task :jekyll do
  rm_rf '_site'
  sh 'jekyll build'
end

desc 'Start the Jekyll server on http://localhost:4000/'
task :server do
  rm_rf '_site'
  puts 'jekyll serve --watch'
  exec 'jekyll serve --watch'
end

CLEAN.include '_site', "_includes/*.html"
CLOBBER.include "_sinatra/sinatra-contrib", "_sinatra"
