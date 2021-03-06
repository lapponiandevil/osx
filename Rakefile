# coding: utf-8
#!/usr/bin/ruby

task :default => [:zshell, :mac_os, :computer_name, :brew, :cask, :configs]
task :continue => [:brew, :cask, :configs]

def curl what
  sh "curl -O #{what}"
end

def brew what
  sh "brew install #{what}"
end

def cask what
  sh "brew cask install #{what}"
end

def in_dir dir
  pwd = Dir.pwd
  begin
    Dir.chdir dir
    yield if block_given?
  ensure
    Dir.chdir pwd
  end
end

def git_config setting, what
  sh "git config --global #{setting} #{what}"
end

def ask_for what
  print what
  STDIN.gets.strip
end

#### Download steps ####

desc "Installs Oh-my zshell"
task :zshell do
  sh "curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh"
end

def install_homebrew
  sh %{/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"} unless \
    Dir.exists? '/usr/local/Homebrew'
  sh "touch ~/.homebrew_analytics_user_uuid && chmod 000 ~/.homebrew_analytics_user_uuid"
end

task :install_homebrew do
  install_homebrew
end

def install_profiles
  #sh "curl -L http://raw.github.com/caiogondim/bullet-train-oh-my-zsh-theme/master/bullet-train.zsh-theme -o ~/.oh-my-zsh/custom/themes/bullet-train.zsh-theme"
  sh "cp bullet-train.zsh-theme ~/.oh-my-zsh/custom/themes/"
  sh "cp .zshrc ~"
end

task :install_profiles do
  install_profiles
end

def set_default_shell
  puts "Changing default shell to ZSH"
  sh "chsh -s $(which zsh)"
end

task :set_default_shell do
  set_default_shell
end

desc "Sets some macOS preferred settings"
task :mac_os do
  sh "git clone https://github.com/lapponiandevil/macos_bootstrap.git"
  in_dir "macos_bootstrap" do
    install_homebrew
    install_profiles
    set_default_shell
  end
end

desc "Updates, upgrades and installs homebrew packages"
task :brew do
  sh "brew update"
  sh "brew upgrade"
  sh "brew cleanup"
  sh "brew tap homebrew/cask"
  sh "brew tap homebrew/cask-fonts"
  sh "brew tap caskroom/versions"
  packages = %w|
    autoconf
    automake
    colordiff
    ctags
    dive
    editorconfig
    fzf
    go
    kubernetes-helm
    jq
    kubectl
    libtool
    libuv
    ngrep
    nmap
    nvm
    postgresql
    pyenv
    rbenv
    readline
    stern
    tree
    ucspi-tcp
    yarn
    watch
    zlib
  |.join(' ')
  brew packages

  sh "/usr/local/opt/fzf/install --no-bash --completion --key-bindings"
end

desc "Installs common casks"
task :cask do
  packages = %w|
    iterm2

    1password
    authy
    caffeine
    chromium
    copyclip
    docker
    firefox
    font-awesome-terminal-fonts
    font-fontawesome
    font-monoid
    font-monoid-nerd-font
    font-monoid-nerd-font-mono
    font-roboto
    font-roboto-condensed
    font-roboto-mono
    font-roboto-mono-for-powerline
    font-roboto-slab
    font-robotomono-nerd-font
    font-robotomono-nerd-font-mono
    google-cloud-sdk
    postman
    sketch
    slack
    spectacle
    spotify
    steam
    typora
    visual-studio-code

    android-studio
  |.join(' ')
  cask packages
end

desc "Sets computer name. Asks for input"
task :computer_name do
  # Set computer name (as done via System Preferences → Sharing)
  computer_name = ask_for "Computer name: "
  sh "sudo scutil --set ComputerName '#{computer_name}'"
  sh "sudo scutil --set HostName '#{computer_name}'"
  sh "sudo scutil --set LocalHostName '#{computer_name}'"
  sh "sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.smb.server NetBIOSName -string '#{computer_name.upcase}'"
  sh "sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist"
end

desc "Configure MacOS settings"
task :macos_config do
  sh "./config-macos.sh"
end

desc "Configure the installed casks"
task :cask_configs do
  sh "mkdir -p ~/.iterm && cp com.googlecode.iterm2.plist ~/.iterm"

  # https://github.com/eczarny/spectacle/issues/244
  sh %{cp spectacle.json "#{ENV['HOME']}/Library/Application Support/Spectacle/Shortcuts.json"}
  sh %{mkdir -p "#{ENV['HOME']}/Library/Application Support/Code/User"}
  sh %{cp vscode.json "#{ENV['HOME']}/Library/Application Support/Code/User/settings.json"}
end

desc 'Configure vim'
task :vim_config do
  sh "cp .vimrc ~/.vimrc"
end

desc "Sets minimum git config. Asks for input"
task :git_config do
  sh "cp .gitconfig ~/.gitconfig"
  git_config "core.editor", "/usr/bin/vim"
  git_config "push.default", "simple"

  user = ask_for "Git user name: "
  git_config "user.name", "'#{user}'"
  email = ask_for "Git user email: "
  git_config "user.email", "'#{email}'"
end

desc 'Configures SSH and its agent'
task :ssh_config do
  sh "mkdir -p ~/.ssh"
  begin
    puts "Please ensure you have 1Password synchronised and are ready to input ssh-key passphrases"
  rescue
    puts "Cancelled."
  ensure
    puts "Done."
  end
end

desc 'Remind user to run NVM installation script'
task :nvm_reminder do
  puts "Please run the script './macos_bootstrap/nvm-install-lts.sh' to complete the installation."
end

desc "Configure Casks, Vim, Git, ssh and nvm"
task :configs do
  in_dir "macos_bootstrap" do
    %w| macos_config
        cask_configs
        vim_config
        git_config
        ssh_config
        nvm_reminder
    |.each do |t|
      Rake::Task[t].invoke()
    end
  end
end
