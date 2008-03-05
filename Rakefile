# -*- ruby -*-
# Rakefile: build ruby auges bindings
#
# Copyright (C) 2008 Red Hat, Inc.
#
# Distributed under the GNU Lesser General Public License v2.1 or later.
# See COPYING for details
#
# Bryan Kearney <bkearney@redhat.com>

require 'rake/clean'
require 'rake/rdoctask'
require 'rake/testtask'
require 'rake/gempackagetask'

PKG_NAME='ruby-augeas'
GEM_NAME='augeas'
PKG_VERSION='0.0.1'
EXT_CONF='ext/augeas/extconf.rb'
MAKEFILE="ext/augeas/Makefile"
AUGEAS_MODULE="ext/augeas/_augeas.so"
SPEC_FILE="ruby-augeas.spec"
AUGEAS_SRC=AUGEAS_MODULE.gsub(/.so$/, ".c")


#
# Building the actual bits
# 
CLEAN.include "**/*~"

CLOBBER.include [ "config.save",
                  "ext/**/*.o", AUGEAS_MODULE,
                  "ext/**/depend", "ext/**/mkmf.log", 
                  MAKEFILE ]
                  
file MAKEFILE => EXT_CONF do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "ruby #{File::basename(EXT_CONF)}"
             $stderr.puts "Failed to run extconf"
             break
         end
    end
end 
file AUGEAS_MODULE => [ MAKEFILE, AUGEAS_SRC ] do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "make"
             $stderr.puts "make failed"
             break
         end
     end
end
desc "Build the native library"
task :build => AUGEAS_MODULE      

#
# Testing
#          
Rake::TestTask.new(:test) do |t|
    t.test_files = FileList['tests/tc_*.rb']
    t.libs = [ 'lib', 'ext/augeas' ]
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
  "Rakefile", "COPYING","README.rdoc",
  "ext/**/*.[ch]", "lib/**/*.rb", "ext/**/MANIFEST", "ext/**/extconf.rb",
  "tests/**/*",
  "spec/**/*"
]

DIST_FILES = FileList[
  "pkg/*.rpm",  "pkg/*.gem",  "pkg/*.zip", "pkg/*.tgz"
]

SPEC = Gem::Specification.new do |s|
    s.name = GEM_NAME
    s.version = PKG_VERSION
    s.email = "augeas-devel@redhat.com"
    s.homepage = "http://augeas.net/"
    s.summary = "Ruby bindings for augeas"
    s.files = PKG_FILES
    s.autorequire = "augeas"
    s.required_ruby_version = '>= 1.8.1'
    s.extensions = "ext/augeas/extconf.rb"
    s.description = "Provides bindings for augeas."
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


