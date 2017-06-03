require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'

desc "little helper bots"

def write_file(content, title, directory, filename)
  File.write("#{directory}/#{filename}", content)
  puts "#{filename} was created in '#{directory}'."
end

def create_file(directory, filename, content, title, editor)
  if File.exists?("#{directory}/#{filename}")
    raise "The file already exists."
  else
    write_file(content, title, directory, filename)
    if editor && !editor.nil?
      sleep 1
      execute("#{editor} #{directory}/#{filename}")
    end
  end
end

# Get the "open" command
def open_command
  if RbConfig::CONFIG["host_os"] =~ /mswin|mingw/
    "start"
  elsif RbConfig::CONFIG["host_os"] =~ /darwin/
    "open"
  else
    "xdg-open"
  end
end


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
    system "mv _site/* #{tmp}"
    system "git checkout -B gh-pages"
    system "rm -rf *"
    system "mv #{tmp}/* ."
    message = "Site updated at #{Time.now.utc}"
    system "git add ."
    system "git commit -am #{message.shellescape}"
    system "git push origin gh-pages --force"
    system "git checkout master"
    system "echo yolo"
  end
end

task :default => :publish

task :make_thumbnails do
  Dir["_paintings/*/*.png"].each do |image|
    puts "working on: " + image
    image_name= File.basename(image).sub(".png","")
    image_path =  (Pathname.new(image)).parent.to_s
    
    thumbnail_path =  image_path+ "/" + image_name + "-thumbnail.jpg"
    square_path =  image_path+ "/" + image_name + "-square.jpg"
    full_path=  image_path+ "/" + image_name + ".jpg"

    command_thumbnail = "convert " + image + " -resize 300 " + thumbnail_path
    command_square = "convert " + image + " -resize 500x500 " + square_path
    command_1200 = "convert " + image + " -resize 1200 " + full_path
    
    sh command_thumbnail unless File.exists?(thumbnail_path)
    sh command_square unless File.exists?(square_path)
    sh command_1200 unless File.exists?(full_path)
  end
  
end

task :paintings_to_posts, :tags do  |t,args|
  t = Time.now
   postdir = "_paintings/" # put the post in the category subdir
  Dir.mkdir(postdir) unless File.exists?(postdir) # make the dir unless it exists
  
  
  Dir["_paintings/*/*.jpg"].each do |image|
    contents = "" # otherwise using it below will be badly scoped
    File.open("_drafts/yyyy-mm-dd-title.md", "rb") do |f|
      contents = f.read
    end
    pn = Pathname.new(image)
    currentDirectory = File.basename(pn.parent.to_s)
    
    imageBaseURL = "/paintings/"+currentDirectory +"/"
    imagename = File.basename(image)
    puts "working on: " + imagename
  
    contents = contents.sub("%date%", t.strftime("%Y-%m-%d %H:%M:%S %z")).sub("%title%", imagename)
    contents = contents.gsub("%category%", "painting")
    contents = contents.sub("%image%", imageBaseURL+imagename)
    contents = contents.sub("%thumbnail%", (imageBaseURL+imagename).sub(".jpg", '') + "-thumbnail.jpg")
    contents = contents.sub("%square_thumbnail%", (imageBaseURL+imagename).sub(".jpg", '') + "-squarethumb.jpg")
    contents = contents.sub("%tags%", args[:tags].gsub("\ ", "\,\ "))
    contents = contents.sub("%body%", "<figure class=\"fullwidth\"><img src=\""+imageBaseURL+imagename +"\" alt=\"A painting titled: " +imagename + " by painter Kyle Cunningham\" /><figcaption>"+imagename +"</figcaption></figure>" )

    filename =File.expand_path("..", image) + "/" + t.strftime("%Y-%m-%d-") + imagename.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-').sub("\.jpg", '') + '.md'
    puts "writing: " + filename 
    
    if File.exists? filename then
      puts "Post already exists: #{filename}"
      return
    end
    File.open(filename, "wb") do |f|
      f.write contents
    end
    puts "created #{filename}"


  end
end




task :images_to_post, :tags do  |t,args|
  t = Time.now
   postdir = "_paintings/automagick" # put the post in the category subdir
  Dir.mkdir(postdir) unless File.exists?(postdir) # make the dir unless it exists
  
  
  Dir["allTheJpg/*.jpg"].each do |image|
    contents = "" # otherwise using it below will be badly scoped
    File.open("_drafts/yyyy-mm-dd-title.md", "rb") do |f|
      contents = f.read
    end

    imagename = File.basename(image)
  
    contents = contents.sub("%date%", t.strftime("%Y-%m-%d %H:%M:%S %z")).sub("%title%", imagename)
    contents = contents.gsub("%category%", "painting")
    contents = contents.sub("%image%", "/allTheJpg/"+imagename)
    contents = contents.sub("%thumbnail%", "/allTheJpg/"+imagename)
    contents = contents.sub("%tags%", args[:tags].gsub("\ ", "\,\ "))
    contents = contents.sub("%body%", "![A painting titled \""+imagename+"\" by painter Kyle Cunningham](/allTheJpg/"+imagename+ ")")

    filename = postdir + "/" + t.strftime("%Y-%m-%d-") + imagename.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-').sub("\.jpg", '') + '.md'
    
    if File.exists? filename then
      puts "Post already exists: #{filename}"
      return
    end
    File.open(filename, "wb") do |f|
      f.write contents
    end
    puts "created #{filename}"


  end
end
