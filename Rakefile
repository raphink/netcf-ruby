# -*- ruby -*-
# Rakefile: build ruby auges bindings
#
# Copyright (C) 2012, Camptocamp
#
# Distributed under the GNU Lesser General Public License v2.1 or later.
# See COPYING for details

require 'rake/clean'
require 'rake/rdoctask'
require 'rake/testtask'
require 'rake/gempackagetask'

PKG_NAME='ruby-netcf'
GEM_NAME=PKG_NAME
PKG_VERSION='0.0.1'
EXT_CONF='ext/netcf/extconf.rb'
MAKEFILE="ext/netcf/Makefile"
NETCF_MODULE="ext/netcf/_netcf.so"
SPEC_FILE="ruby-netcf.spec"
NETCF_SRC=NETCF_MODULE.gsub(/.so$/, ".c")

#
# Building the actual bits
#
CLEAN.include [ "**/*~",
                "ext/**/*.o", NETCF_MODULE,
                "ext/**/depend" ]

CLOBBER.include [ "config.save",
                  "ext/**/mkmf.log",
                  MAKEFILE ]

file MAKEFILE => EXT_CONF do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "ruby #{File::basename(EXT_CONF)}"
             $stderr.puts "Failed to run extconf"
             break
         end
    end
end
file NETCF_MODULE => [ MAKEFILE, NETCF_SRC ] do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "make"
             $stderr.puts "make failed"
             break
         end
     end
end
desc "Build the native library"
task :build => NETCF_MODULE

#
# Testing
#
Rake::TestTask.new(:test) do |t|
    t.test_files = FileList['tests/tc_*.rb']
    t.libs = [ 'lib', 'ext/netcf' ]
end
task :test => :build


#
# Generate the documentation
#
Rake::RDocTask.new do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/site/api"
    rd.rdoc_files.include("README.rdoc", "ext/**/*.[ch]","lib/**/*.rb")
end

#
# Packaging
#
PKG_FILES = FileList[
  "Rakefile", "COPYING","README.rdoc", "NEWS",
  "ext/**/*.[ch]", "lib/**/*.rb", "ext/**/MANIFEST", "ext/**/extconf.rb",
  "tests/**/*",
  "spec/**/*"
]

DIST_FILES = FileList[
  "pkg/*.tgz", "pkg/*.gem"
]

SPEC = Gem::Specification.new do |s|
    s.name = GEM_NAME
    s.version = PKG_VERSION
    s.email = "netcf-devel@redhat.com"
    s.homepage = "https://fedorahosted.org/netcf/"
    s.summary = "Ruby bindings for netcf"
    s.files = PKG_FILES
    s.autorequire = "netcf"
    s.required_ruby_version = '>= 1.8.1'
    s.extensions = "ext/netcf/extconf.rb"
    s.description = "Provides bindings for netcf."
end

Rake::GemPackageTask.new(SPEC) do |pkg|
    pkg.need_tar = true
    pkg.need_zip = true
end

desc "Build (S)RPM for #{PKG_NAME}"
task :rpm => [ :package ] do |t|
    system("sed -e 's/@VERSION@/#{PKG_VERSION}/' #{SPEC_FILE} > pkg/#{SPEC_FILE}")
    Dir::chdir("pkg") do |dir|
        dir = File::expand_path(".")
        system("rpmbuild --define '_topdir #{dir}' --define '_sourcedir #{dir}' --define '_srcrpmdir #{dir}' --define '_rpmdir #{dir}' --define '_builddir #{dir}' -ba #{SPEC_FILE} > rpmbuild.log 2>&1")
        if $? != 0
            raise "rpmbuild failed"
        end
    end
end

desc "Release a version to the site"
task :dist => [ :rpm ] do |t|
    puts "Copying files"
    unless sh "scp -p #{DIST_FILES.to_s} et:/var/www/sites/netcf.et.redhat.com/download/ruby"
        $stderr.puts "Copy to et failed"
        break
    end
    puts "Commit and tag #{PKG_VERSION}"
    system "git commit -a -m 'Released version #{PKG_VERSION}'"
    system "git tag -s -m 'Tag release #{PKG_VERSION}' release-#{PKG_VERSION}"
end

task :sync do |t|
    system "rsync -rav doc/site/ et:/var/www/sites/netcf.et.redhat.com/docs/ruby/"
end
