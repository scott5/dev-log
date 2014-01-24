Installing on an Amazon EC2 t1.micro instance running Ubuntu Server 12.04.3 LTS - ami-a73264ce (64-bit)

### General EC2 Ubuntu fixups ###

    sudo apt-get update
    sudo apt-get install build-essential git zsh
    sudo -s
    chsh -s /bin/zsh ubuntu # change the shell to Zsh for the default user, ubuntu

Log out, then back in again.

    curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh
    curl -o ~/.vimrc https://raw.github.com/pingswept/linux-tool-configs/master/vimrc

Also, copy gitconfig from https://github.com/pingswept/linux-tool-configs

### Install Node ###

For Ghost, need a newer version of Node: Required: {"node":">= 0.6.13 < 0.11.0"}

    wget http://nodejs.org/dist/v0.10.24/node-v0.10.24.tar.gz
    tar xzf node-v0.10.24.tar.gz
    cd node-v0.10.24
    ./configure
    make
    sudo make install
    node --version
        v0.10.24

### Install Ghost ###

First, we need the Bourbon gem, which requires rubygems.

    sudo apt-get install rubygems
    echo "gem: --no-ri --no-rdoc" > ~/.gemrc

Then, install Bourbon gem.

    sudo gem install bourbon
    Fetching: sass-3.2.13.gem (100%)
    Fetching: thor-0.18.1.gem (100%)
    Fetching: bourbon-3.1.8.gem (100%)
    Successfully installed sass-3.2.13
    Successfully installed thor-0.18.1
    Successfully installed bourbon-3.1.8
    3 gems installed

Install Bundler gem.

    sudo gem install bundler

Then, clone and do some more stuff.

(Here, we're cloning from Github, which is stupid, but it's the only way to get the pages feature right now. In the future, should use Ghost 0.4.)

    wget https://codeload.github.com/TryGhost/Ghost/tar.gz/0.4.0
    tar xzvf 0.4.0
    
    sudo npm install -g grunt-cli
    npm install

Use grunt to build CSS and JS files

    grunt init
    Running "shell:bourbon" (shell) task
    
    Running "update_submodules" task
    
    Running "sass:compress" (sass) task
    File "./core/client/assets/css/screen.css" created.
    
    Running "handlebars:core" (handlebars) task
    File "core/client/tpl/hbs-tpl.js" created.
    
    Running "concat:dev" (concat) task
    File "core/built/scripts/vendor.js" created.
    File "core/built/scripts/helpers.js" created.
    File "core/built/scripts/templates.js" created.
    File "core/built/scripts/models.js" created.
    File "core/built/scripts/views.js" created.
    
    Running "concat:prod" (concat) task
    File "core/built/scripts/ghost.js" created.
    
    Done, without errors.

Actually start ghost:

    npm start # from within ghost directory

Need to open a port for web serving on EC2 instance before the site will be visible externally.

http://localhost:2368/ghost/

### Install supervisor ###

    sudo apt-get install supervisor

Add Ghost conf file so that Supervisor runs Ghost on startup.

    sudo vim /etc/supervisor/conf.d/ghost.conf

    [program:ghost]
    command = node /home/ubuntu/ghost-696cfe7018/index.js
    directory = /home/ubuntu/ghost-696cfe7018
    user = ubuntu
    autostart = true
    autorestart = true
    stdout_logfile = /var/log/supervisor/ghost.log
    stderr_logfile = /var/log/supervisor/ghost_err.log
    environment = NODE_ENV="production"

    sudo /etc/init.d/supervisor stop
    sudo /etc/init.d/supervisor start
    
Rather than starting and stopping Supervisorm probably better to do (seems to work, but needs more testing):

    sudo supervisorctl reread