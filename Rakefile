# -*- ruby -*-
#
# Copyright (C) 2012  Kouhei Sutou <kou@cozmixng.org>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

require "rubygems/remote_fetcher"
require "rake/clean"

$LOAD_PATH.unshift(File.join(File.dirname(__FILE__), "lib"))

require "generator"

task :default => :generate

namespace :images do
  generated_images = []
  Dir.glob("assets/images/go-*.svg").each do |go_svg|
    go_mini_png = go_svg.gsub(/\.svg/, "-mini.png")
    file go_mini_png => go_svg do |task|
      sh("inkscape",
         "--export-png=#{go_mini_png}",
         "--export-width=16",
         "--export-height=16",
         "--export-background-opacity=0",
         go_svg)
    end
    generated_images << go_mini_png
  end
  CLOBBER.concat(generated_images)
  task :generate => generated_images
end

desc "Generate HTML"
task :generate => "images:generate" do
  generator = Generator.new(ENV["HTML_DIR"] || "html")
  generator.generate
end

@gems_dir = "gems"

def download_gem(spec, source_uri=nil)
  source_uri ||= Gem.sources.first

  if spec.respond_to?(:cache_file)
    gem_base_name = File.basename(spec.cache_file)
  else
    gem_base_name = spec.file_name
  end
  gem_path = File.join(@gems_dir, gem_base_name)
  return if File.exist?(gem_path)

  tmp_dir = "tmp"
  rm_rf(tmp_dir)
  mkdir_p(tmp_dir)

  remote_fetcher = Gem::RemoteFetcher.fetcher
  remote_fetcher.download(spec, source_uri, tmp_dir)
  downloaded_gem_path = File.join(tmp_dir, "cache", gem_base_name)

  mkdir_p(@gems_dir)
  mv(downloaded_gem_path, gem_path)

  rm_rf(tmp_dir)
end

namespace :gems do
  desc "Fetch all slide gems"
  task :fetch do
    dependency = Gem::Deprecate.skip_during do
      Gem::Dependency.new(/\Arabbit-slide-/)
    end
    spec_fetcher = Gem::SpecFetcher.fetcher
    spec_and_sources = spec_fetcher.fetch(dependency)
    spec_and_sources.each do |spec, source_uri|
      download_gem(spec, source_uri)
    end
  end

  desc "Update existing slide gems"
  task :update do
    updated_gems = {}
    Dir.glob(File.join(@gems_dir, "*.gem")).each do |gem_path|
      format = Gem::Format.from_file_by_path(gem_path.to_s)
      spec = format.spec
      next if updated_gems.has_key?(spec.name)
      download_gem(spec)
      updated_gems[spec.name] = true
    end
  end
end
