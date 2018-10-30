# Rakefile for llmk.
# Public domain.

require 'rake/clean'
require 'pathname'

# basics
LLMK_VERSION = "0.0.0"
PKG_NAME = "llmk-#{LLMK_VERSION}"

# woking/temporaly dirs
PWD = Pathname.pwd
TMP_DIR = PWD + "tmp"

# options for ronn
OPT_MAN = "--manual=\"llmk manual\""
OPT_ORG = "--organization=\"llmk #{LLMK_VERSION}\""

# cleaning
CLEAN.include(["doc/*", "tmp"])
CLEAN.include(["*.log", "*.synctex.gz", "*.dvi", "*.pdf", "*.aux"])
CLEAN.exclude(["doc/*.md", "doc/*.tex", "doc/*.pdf"])
CLOBBER.include(["doc/*.pdf", "*.zip"])
CLEAN.exclude(["doc/logo.png"])

desc "Run tests (only listed specs, if specified)"
task :test do |task, args|
  # run rspec
  args = args.to_a
  if args.size > 0
    f_list = args.map{|f| "spec/#{f}_spec.rb"}.join(" ")
    sh "bundle exec rspec #{f_list}"
  else
    sh "bundle exec rspec"
  end
end

desc "Generate all documentation"
task :doc do
  cd "doc"
  sh "llmk -q llmk.tex > #{File::NULL} 2> #{File::NULL}"
  sh "bundle exec ronn -r #{OPT_MAN} #{OPT_ORG} llmk.1.md 2> #{File::NULL}"
end

desc "Preview the manpage"
task :man do
  cd "doc"
  sh "bundle exec ronn -m #{OPT_MAN} #{OPT_ORG} llmk.1.md"
end

desc "Setup TeX Live on Travis CI"
task :setup_travis do
  # judge platform
  fail "This task only works on Travis CI" if not platform = ENV["TRAVIS_OS_NAME"]

  # install TeX Live if the cached version of TeX Live is not available
  if not system("which texlua > #{File::NULL} 2> #{File::NULL}")
    puts "* Installing TeX Live"

    # prepare the install dir
    HOME = ENV["HOME"]
    INSTALL_DIR = TMP_DIR + Time.now.strftime("%F")
    mkdir_p INSTALL_DIR
    cd INSTALL_DIR

    # download install-tl
    sh "wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz"
    sh "tar zxvf install-tl-unx.tar.gz"
    cd Dir.glob("install-tl-2018*")[0]

    # config
    profile = <<~EOF
      selected_scheme scheme-small
      TEXDIR #{HOME}/texlive
      TEXMFCONFIG #{HOME}/.texlive/texmf-config
      TEXMFHOME #{HOME}/texmf
      TEXMFLOCAL #{HOME}/texlive/texmf-local
      TEXMFSYSCONFIG #{HOME}/texlive/texmf-config
      TEXMFSYSVAR #{HOME}/texlive/texmf-var
      TEXMFVAR #{HOME}/.texlive/texmf-var
      option_doc 0
      option_src 0
    EOF

    if platform == "osx"
      File.open("texdoc.profile", "w") {|f| f.puts(profile + "binary_x86_64-darwin 1")}
    else
      File.open("texdoc.profile", "w") {|f| f.puts(profile + "binary_x86_64-linux 1")}
    end

    # run install script
    opt_profile = "-profile ./texdoc.profile"
    opt_repo = "-repository http://ctan.mirror.rafal.ca/systems/texlive/tlnet"
    sh "./install-tl #{opt_profile} #{opt_repo}"
    sh "tlmgr init-usertree"
    sh "tlmgr #{opt_repo} install collection-langjapanese"

    # finish
    cd PWD
    rm_rf INSTALL_DIR
  end
end

desc "Setup TeX Live on AppVeyor"
task :setup_appveyor do
  # judge platform
  fail "This task only works on AppVeyor" if not ENV["APPVEYOR"]

  # install TeX Live if the cached version of TeX Live is not available
  if not system("which texlua > #{File::NULL} 2> #{File::NULL}")
    puts "* Installing TeX Live"

    # prepare the install dir
    INSTALL_DIR = TMP_DIR + Time.now.strftime("%F")
    mkdir_p INSTALL_DIR
    cd INSTALL_DIR

    # download install-tl
    sh "curl -O http://ctan.mirror.rafal.ca/systems/texlive/tlnet/install-tl.zip"
    sh "unzip install-tl.zip"
    cd Dir.glob("install-tl-2018*")[0]

    # config
    profile = <<~EOF
      selected_scheme scheme-small
      TEXDIR /texlive
      TEXMFCONFIG /.texlive/texmf-config
      TEXMFHOME /texmf
      TEXMFLOCAL /texlive/texmf-local
      TEXMFSYSCONFIG /texlive/texmf-config
      TEXMFSYSVAR /texlive/texmf-var
      TEXMFVAR /.texlive/texmf-var
      binary_win32 1
      option_doc 0
      option_src 0
    EOF

    File.open("texdoc.profile", "w") {|f| f.puts(profile)}

    # run install script
    opt_profile = "-profile ./texdoc.profile"
    opt_repo = "-repository http://ctan.mirror.rafal.ca/systems/texlive/tlnet"
    sh "echo y | install-tl-windows.bat #{opt_profile} #{opt_repo}"
    sh "tlmgr.bat init-usertree"
    sh "tlmgr.bat #{opt_repo} install collection-langjapanese"

    # finish
    cd PWD
    rm_rf INSTALL_DIR
  end
end
