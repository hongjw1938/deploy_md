배포수업



1. aws에서 EC2에서 다음과 같이 설정하도록한다.

- 인스턴스 시작 - AWS LINUX2 AMI - 프리티어 사용가능한 것으로 선택(두 번째꺼 했어야했는데 강사님이 첫 번째꺼해서 우리는 https를 하지못한다...), 다음 인스턴스 세부정보 구성으로 -> 보안그룹까지 건드릴 것 없이 쭉 넘어가고, 시작까지 쭉 눌러줍시다. 
- 새 키 페어 생성하고 키페어 다운로드 시켜준다.(.pem)

1. c9에서 .gitignore파일을 건드려줍시다.

- gitignore.io에 들어가 다음과 같이 쓰고, create눌러준다.



만들어진 파일은 c9의 .gitignore에  /db/*.sqlite3-journal까지 덮어씌워준다. 그 아래로는 덮어씌우면 안된다.
그리고 .gitignore에서 gemfile lock~, config/secret.yml으로 시작하는 두 부분의 주석을 풀어준다.
그다음 다음과 같이 추가해주도록하자

    # assets폴더 밑에 있는거 다 날려준다. 왜냐면 precomplie할때마다 삭제되고 또 충돌나기때문에.
    /public/assets/*
    
    *.pem

그 후 public/assets를 날려주도록 하자

    namkun:~/workspace (master) $ rake assets:clobber
    WARNING: Use strings for Figaro configuration. 563215 was converted to "563215".
    I, [2018-07-24T01:24:14.052072 #54598]  INFO -- : Removed /home/ubuntu/workspace/public/assets

그후 root에 pem키를 넣어주고 다음과 같이 실행한다.

    namkun:~/muckbo (master) $ cd
    namkun:~ $ ls
    lib/  muckbo/  muckbo.pem  workspace/

그 후 aws에서 ec2의 탄력적IP를 생성, 그리고 오른쪽클릭후 주소연결을 누른뒤, 인스턴스를 선택하고, 프라이빗은 선택x -> 어소시에이트해준다.
탄력적 IP를 생성하여, 서버를 껐다가 켜도 IPv4 퍼블릭 IP 가 변경되지 않도록 하기 위함.
그 다음 c9 console에서 다음과 같이 실행한다.

    namkun:~ $ ssh -i muckbo.pem ec2-user@[IPv4 퍼블릭 IP]
    The authenticity of host '13.209.95.226 (13.209.95.226)' can't be established.
    ECDSA key fingerprint is 84:6a:2e:09:e7:0d:3d:3a:85:63:df:8e:32:a6:b1:54.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '13.209.95.226' (ECDSA) to the list of known hosts.
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0644 for 'muckbo.pem' are too open.
    It is required that your private key files are NOT accessible by others.
    This private key will be ignored.
    bad permissions: ignore key: muckbo.pem
    Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

왜 denied가 떴는가? -> pem파일은 아주 중요한데 권한이 너무 열려있어서 그럼

다음과 같이 console에 입력하여 권한을 바꿔주도록하자.

    namkun:~ $ sudo chmod 400 muckbo.pem

그러고 다시 입력해보면 접속이 된다.

    namkun:~ $ ssh -i muckbo.pem ec2-user@13.209.95.226
    
           __|  __|_  )
           _|  (     /   Amazon Linux 2 AMI
          ___|\___|___|
    
    https://aws.amazon.com/amazon-linux-2/
    No packages needed for security; 4 packages available
    Run "sudo yum update" to apply all updates.
    [ec2-user@ip-172-31-20-132 ~]$ 

그 후 sudo yum update를 통해서 업데이트 해주도록하자

    [ec2-user@ip-172-31-20-132 ~]$ sudo yum update

git -v로 git이 설치되어있는지 확인하고, 없으면 다음과 같은 명령어로 설치해주도록 하자

    [ec2-user@ip-172-31-20-132 ~]$ git -v
    -bash: git: command not found
    [ec2-user@ip-172-31-20-132 ~]$ sudo yum install git-all.noarch

설치 하고 나면 이렇게 보입니다.

    [ec2-user@ip-172-31-20-132 ~]$ git --version
    git version 2.14.4

그 뒤, 다음과 같이 명령어를 실행시켜줍니다.(ruby 2.4.4 버전 기준)

    [ec2-user@ip-172-31-20-132 ~]$ cd
    [ec2-user@ip-172-31-20-132 ~]$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    Cloning into '/home/ec2-user/.rbenv'...
    remote: Counting objects: 2737, done.
    remote: Total 2737 (delta 0), reused 0 (delta 0), pack-reused 2737
    Receiving objects: 100% (2737/2737), 513.94 KiB | 1.20 MiB/s, done.
    Resolving deltas: 100% (1715/1715), done.
    [ec2-user@ip-172-31-20-132 ~]$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
    [ec2-user@ip-172-31-20-132 ~]$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
    [ec2-user@ip-172-31-20-132 ~]$ exec $SHELL

정상적으로 rbenv가 설치됬는지 확인하는 법은 다음과 같다.

    [ec2-user@ip-172-31-20-132 ~]$ rbenv
    rbenv 1.1.1-37-g1c772d5
    Usage: rbenv <command> [<args>]
    
    Some useful rbenv commands are:
       commands    List all available rbenv commands
       local       Set or show the local application-specific Ruby version
       global      Set or show the global Ruby version
       shell       Set or show the shell-specific Ruby version
       rehash      Rehash rbenv shims (run this after installing executables)
       version     Show the current Ruby version and its origin
       versions    List all Ruby versions available to rbenv
       which       Display the full path to an executable
       whence      List all Ruby versions that contain the given executable
    
    See `rbenv help <command>' for information on a specific command.
    For full documentation, see: https://github.com/rbenv/rbenv#readme

이렇게 나온다면 정상적으로 설치가 잘된 것.

설치가 잘 된것을 확인했다면, 다음과 같이 코드를 넣어준다.

    [ec2-user@ip-172-31-20-132 ~]$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
    Cloning into '/home/ec2-user/.rbenv/plugins/ruby-build'...
    remote: Counting objects: 8929, done.
    remote: Compressing objects: 100% (11/11), done.
    remote: Total 8929 (delta 1), reused 4 (delta 0), pack-reused 8918
    Receiving objects: 100% (8929/8929), 1.87 MiB | 1.91 MiB/s, done.
    Resolving deltas: 100% (5739/5739), done.
    [ec2-user@ip-172-31-20-132 ~]$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
    [ec2-user@ip-172-31-20-132 ~]$ exec $SHELL

rbenv 2.4.0을 설치해봅시다.

    [ec2-user@ip-172-31-20-132 ~]$ rbenv install 2.4.0
    Downloading ruby-2.4.0.tar.bz2...
    -> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.bz2
    Installing ruby-2.4.0...
    
    BUILD FAILED (Amazon Linux 2 using ruby-build 20180618-7-gb6428c7)
    
    Inspect or clean up the working tree at /tmp/ruby-build.20180724021836.13217
    Results logged to /tmp/ruby-build.20180724021836.13217.log
    
    Last 10 log lines:
    checking for ruby... false
    checking build system type... x86_64-pc-linux-gnu
    checking host system type... x86_64-pc-linux-gnu
    checking target system type... x86_64-pc-linux-gnu
    checking for gcc... no
    checking for cc... no
    checking for cl.exe... no
    configure: error: in `/tmp/ruby-build.20180724021836.13217/ruby-2.4.0':
    configure: error: no acceptable C compiler found in $PATH
    See `config.log' for more details

이 오류는 gcc(c컴파일러)가 없기에 발생하는 에러이기에

    [ec2-user@ip-172-31-20-132 ~]$ sudo yum groupinstall "Development Tools"

이 명령어로 development tools를 설치하여 gcc라는 c 컴파일러를 설치하도록하자.

그 후 다시 rbenv install 2.4.0을 다시 해주도록 하자.

    [ec2-user@ip-172-31-20-132 ~]$ rbenv install 2.4.0
    Downloading ruby-2.4.0.tar.bz2...
    -> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.bz2
    Installing ruby-2.4.0...
    
    BUILD FAILED (Amazon Linux 2 using ruby-build 20180618-7-gb6428c7)
    
    Inspect or clean up the working tree at /tmp/ruby-build.20180724022111.28024
    Results logged to /tmp/ruby-build.20180724022111.28024.log
    
    Last 10 log lines:
    installing capi-docs:         /home/ec2-user/.rbenv/versions/2.4.0/share/doc/ruby
    The Ruby openssl extension was not compiled.
    The Ruby readline extension was not compiled.
    ERROR: Ruby install aborted due to missing extensions
    Try running `yum install -y openssl-devel readline-devel` to fetch missing dependencies.
    
    Configure options used:
      --prefix=/home/ec2-user/.rbenv/versions/2.4.0
      LDFLAGS=-L/home/ec2-user/.rbenv/versions/2.4.0/lib 
      CPPFLAGS=-I/home/ec2-user/.rbenv/versions/2.4.0/include 

또 오류가 난다...하라는대로 하자

    [ec2-user@ip-172-31-20-132 ~]$ sudo yum install -y openssl-devel readline-devel

다시 rbenv...성공!

    [ec2-user@ip-172-31-20-132 ~]$ rbenv install 2.4.0
    Downloading ruby-2.4.0.tar.bz2...
    -> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.bz2
    Installing ruby-2.4.0...
    Installed ruby-2.4.0 to /home/ec2-user/.rbenv/versions/2.4.0

그 후 설치가 성공적으로 잘 되었는지 확인해본다.

    [ec2-user@ip-172-31-20-132 ~]$ rbenv global 2.4.0
    [ec2-user@ip-172-31-20-132 ~]$ rbenv rehash -> 설치한 것을 적용한다라는 의미
    [ec2-user@ip-172-31-20-132 ~]$ ruby -v
    ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-linux]

여기까지가 배포의 20%..

이제 우리가 했던 프로젝트를 가져와보자

    [ec2-user@ip-172-31-20-132 ~]$ ls
    [ec2-user@ip-172-31-20-132 ~]$ git clone https://github.com/namekun/Muckbo_finish.git
    Cloning into 'Muckbo_finish'...
    remote: Counting objects: 733, done.
    remote: Compressing objects: 100% (393/393), done.
    remote: Total 733 (delta 338), reused 693 (delta 298), pack-reused 0
    Receiving objects: 100% (733/733), 3.85 MiB | 2.44 MiB/s, done.
    Resolving deltas: 100% (338/338), done.
    [ec2-user@ip-172-31-20-132 ~]$ ls
    Muckbo_finish // 현재 내 프로젝트가 잘 들어와 있다.

gem 들을 설치해보자.

    [ec2-user@ip-172-31-20-132 ~]$ gem install bundler
    Fetching: bundler-1.16.3.gem (100%)
    Successfully installed bundler-1.16.3
    Parsing documentation for bundler-1.16.3
    Installing ri documentation for bundler-1.16.3
    Done installing documentation for bundler after 4 seconds
    1 gem installed

저렇게 하고, 경로를 바꾼뒤 , gem을 설치한다.

    [ec2-user@ip-172-31-20-132 ~]$ cd Muckbo_finish/
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ bundle install

SQLite3 라는 gem에서 오류가 날것.

    An error occurred while installing sqlite3 (1.3.13), and Bundler
    cannot continue.
    Make sure that `gem install sqlite3 -v '1.3.13' --source
    'https://rubygems.org/'` succeeds before bundling.
    
    In Gemfile:
      sqlite3
    

위로 올라가서보면

    sqlite3.h is missing. Try 'brew install sqlite3',
    'yum install sqlite-devel' or 'apt-get install libsqlite3-dev'
    and check your shared library search path (the
    location where your sqlite3 shared library is located).

라고 써있는데, 이중 centOS는 yum이니까 저걸로 설치해주자

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo yum install sqlite-devel

설치가 완료되면 다시 bundle install을 해준다. 그럼 잘될것.

passenger라는 gem을 설치한다 (nginx를 쉽게 설치하도록 도와주는 gem)

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ gem install passenger

이제 중요합니다.

패스워드를 설정해주도록 합니다. 

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo passwd
    Changing password for user root.
    New password: 평소쓰는것 했습니다...
    Retype new password: 
    passwd: all authentication tokens updated successfully.

루트 권한으로 들어간다.

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ cd
    [ec2-user@ip-172-31-20-132 ~]$ su
    Password: 
    [root@ip-172-31-20-132 ec2-user]# 

nginx를 설치해주자

    [root@ip-172-31-20-132 ec2-user]# passenger-install-nginx-module

    [root@ip-172-31-20-132 ec2-user]# passenger-install-nginx-module
    Welcome to the Phusion Passenger Nginx module installer, v5.3.3.
    
    This installer will guide you through the entire installation process. It
    shouldn't take more than 5 minutes in total.
    
    Here's what you can expect from the installation process:
    
     1. This installer will compile and install Nginx with Passenger support.
     2. You'll learn how to configure Passenger in Nginx.
     3. You'll learn how to deploy a Ruby on Rails application.
    
    Don't worry if anything goes wrong. This installer will advise you on how to
    solve any problems.
    
    Press Enter to continue, or Ctrl-C to abort.
    
    -------------------------------------------
    
    Which languages are you interested in?
    
    Use <space> to select.
    If the menu doesn't display correctly, press '!'
    
       (*)  Ruby
       ( )  Python < - shift + 1 누르고 파이썬에서 space 누른뒤 해제해준다.
       ( )  Node.js
     > ( )  Meteor

이렇게 하고 enter 누르면 설치가 안될 것. 그러면 이제 이렇게 권한을 다시 설정해준다.

    [root@ip-172-31-20-132 ec2-user]# sudo chmod o+x "/home/ec2-user"

그러고 다시하면?

    [root@ip-172-31-20-132 ec2-user]# passenger-install-nginx-module
    Welcome to the Phusion Passenger Nginx module installer, v5.3.3.
    
    This installer will guide you through the entire installation process. It
    shouldn't take more than 5 minutes in total.
    
    Here's what you can expect from the installation process:
    
     1. This installer will compile and install Nginx with Passenger support.
     2. You'll learn how to configure Passenger in Nginx.
     3. You'll learn how to deploy a Ruby on Rails application.
    
    Don't worry if anything goes wrong. This installer will advise you on how to
    solve any problems.
    
    Press Enter to continue, or Ctrl-C to abort.
    
    
    --------------------------------------------
    
    Which languages are you interested in?
    
    Use <space> to select.
    If the menu doesn't display correctly, press '!'
    
     > (*)  Ruby
       ( )  Python
       ( )  Node.js
       ( )  Meteor
    
    --------------------------------------------
    
    Checking for required software...
    
     * Checking for C compiler...
          Found: yes
          Location: /usr/bin/cc
     * Checking for C++ compiler...
          Found: yes
          Location: /usr/bin/c++
     * Checking for A download tool like 'wget' or 'curl'...
          Found: yes
          Location: /usr/bin/wget
     * Checking for Curl development headers with SSL support...
          Found: no
          Error: Cannot find the `curl-config` command.
     * Checking for OpenSSL development headers...
          Found: yes
          Location: /usr/include/openssl/ssl.h
     * Checking for Zlib development headers...
          Found: yes
          Location: /usr/include/zlib.h
     * Checking for Rake (associated with /home/ec2-user/.rbenv/versions/2.4.0/bin/ruby)...
          Found: yes
          Location: /home/ec2-user/.rbenv/versions/2.4.0/bin/ruby /home/ec2-user/.rbenv/versions/2.4.0/bin/rake
     * Checking for OpenSSL support for Ruby...
          Found: yes
     * Checking for RubyGems...
          Found: yes
     * Checking for Ruby development headers...
          Found: yes
          Location: /home/ec2-user/.rbenv/versions/2.4.0/include/ruby-2.4.0/ruby.h
     * Checking for rack...
          Found: yes
    
    Some required software is not installed.
    But don't worry, this installer will tell you how to install them.
    Press Enter to continue, or Ctrl-C to abort.
    

짜잔~ curl에서 오류가 납니다~ 설치해봅시다.

    [root@ip-172-31-20-132 ec2-user]# yum install libcurl-devel.x86_64
    ...
    
    Installed:
      libcurl-devel.x86_64 0:7.55.1-12.amzn2.0.1                               
    
    Complete!
    

자.. 다시 install passenger...하면 이런 말이 나옵니다. 

    ...
    Your system does not have a lot of virtual memory
    
    Compiling Phusion Passenger works best when you have at least 1024 MB of virtual
    memory. However your system only has 987 MB of total virtual memory (987 MB
    RAM, 0 MB swap). It is recommended that you temporarily add more swap space
    before proceeding. You can do it as follows:
    
      sudo dd if=/dev/zero of=/swap bs=1M count=1024
      sudo mkswap /swap
      sudo swapon /swap
    
    See also https://wiki.archlinux.org/index.php/Swap for more information about
    the swap file on Linux.
    
    If you cannot activate a swap file (e.g. because you're on OpenVZ, or if you
    don't have root privileges) then you should install Phusion Passenger through
    DEB/RPM packages. For more information, please refer to our installation
    documentation:
    
      https://www.phusionpassenger.com/library/install/nginx/
    
    Press Ctrl-C to abort this installer (recommended).
    Press Enter if you want to continue with installation anyway.
    

대충 너 메모리 없어서 서버 꺼질수도 있다는 경고문

그러니 저기에 있는 명령어들 해봅시다.

    [root@ip-172-31-20-132 ec2-user]# sudo dd if=/dev/zero of=/swap bs=1M count=1024
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 13.6138 s, 78.9 MB/s
    [root@ip-172-31-20-132 ec2-user]# sudo mkswap /swap
    mkswap: /swap: insecure permissions 0644, 0600 suggested.
    Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
    no label, UUID=ed3eb357-86e3-4079-9de8-4e7f6648d10c
    [root@ip-172-31-20-132 ec2-user]# sudo swapon /swap
    swapon: /swap: insecure permissions 0644, 0600 suggested.
    

자 다시 nginx install...

    --------------------------------------------
    
    Automatically download and install Nginx?
    
    Nginx doesn't support loadable modules such as some other web servers do,
    so in order to install Nginx with Passenger support, it must be recompiled.
    
    Do you want this installer to download, compile and install Nginx for you?
    
     1. Yes: download, compile and install Nginx for me. (recommended)
        The easiest way to get started. A stock Nginx 1.14.0 with Passenger
        support, but with no other additional third party modules, will be
        installed for you to a directory of your choice.
    
     2. No: I want to customize my Nginx installation. (for advanced users)
        Choose this if you want to compile Nginx with more third party modules
        besides Passenger, or if you need to pass additional options to Nginx's
        'configure' script. This installer will  1) ask you for the location of
        the Nginx source code,  2) run the 'configure' script according to your
        instructions, and  3) run 'make install'.
    
    Whichever you choose, if you already have an existing Nginx configuration file,
    then it will be preserved.
    
    Enter your choice (1 or 2) or press Ctrl-C to abort: 
    

여기서 1번을 선택해줍니다.

    -------------------------------------
    
    Where do you want to install Nginx to?
    
    Please specify a prefix directory [/opt/nginx]: 
    

경로 선택하라는 말 나오면 그냥 Enter~!

하면 nginx가 install 됩니다. 그러고 enter 누르고

    [root@ip-172-31-20-132 ec2-user]# exit
    exit
    [ec2-user@ip-172-31-20-132 ~]$ 
    

root 권한에서 나옵니다. 그 후 IP V4 Public 을 주소창에 치면 계속 돌아갑니다.

주소를 뚫어(?)봅시다.

aws ec2 대쉬보드에서 보안그룹으로 갑니다. 그 후, 보안그룹을 하나 선택, 인바운드 규칙 편집으로 갑니다.





여기서 해당 아이피로 들어가면 안됨. 설정해주러갑시다.

    [ec2-user@ip-172-31-20-132 ~]$ sudo vi /opt/nginx/conf/nginx.conf
    

이렇게 치면

    #user  nobody;
    worker_processes  1;
    
    #error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;
    
    #pid        logs/nginx.pid;
    
    
    events {
        worker_connections  1024;
    "/opt/nginx/conf/nginx.conf" 120L, 2821C
    ...
    

이렇게 나오는데 여기에서 server{ listen 80; server name localhost;}를 찾아서

        #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
        
        server {
            listen 80;
            passenger_enabled on;
            root
        }
    
        server {
            listen       80;    #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
        
        server {
            listen 80;
            passenger_enabled on;
            root
        }
    
        server {
            listen       80;
     ...
    

이렇게 수정하고 esc + :wq하고 나와서 root에 들어갈 주소를 찾는다.

    [ec2-user@ip-172-31-20-132 ~]$ cd Muckbo_finish/
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ cd public/
    [ec2-user@ip-172-31-20-132 public]$ pwd
    /home/ec2-user/Muckbo_finish/public
    

저 주소를 아까 거기로 가서 root뒤에 붙여준다.

    ...
    server {
            listen 80;
            passenger_enabled on;
            root /home/ec2-user/Muckbo_finish/public;
        }
    ...
    

    [ec2-user@ip-172-31-20-132 public]$ sudo /opt/nginx/sbin/nginx 
    

하고 아까 아이피로 들어가면?



이렇게 나옵니다. 오류가 있다면 (뭔지안알랴줌) 나오는 화면입니다. 서버를 정지합시다.

    [ec2-user@ip-172-31-20-132 public]$ sudo /opt/nginx/sbin/nginx -s stop
    

데이터베이스를 migrate 해봅시다.

    [ec2-user@ip-172-31-20-132 public]$ rake db:migrate
    

하면 ExecJS::RuntimeUnavailable: Could not find a JavaScript runtime. 이라는 에러가 나옵니다.

    [ec2-user@ip-172-31-20-132 ~]$ cd Muckbo_finish/
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ vi Gemfile
    

들어가서 요부분의 주석을 해제하고, 나와서 bundle install.

    # Gem rubyRacer
    

그리고 rake db:migrate

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ rake db:migrate
    == 20180709072029 CreateRooms: migrating ======================================
    -- create_table(:rooms)
       -> 0.0012s
    == 20180709072029 CreateRooms: migrated (0.0014s) =============================
    
    == 20180712113533 CreateRoomsTags: migrating ==================================
    -- create_table(:rooms_tags, {:id=>false})
       -> 0.0019s
    == 20180712113533 CreateRoomsTags: migrated (0.0021s) =========================
    
    == 20180712114410 CreateTags: migrating =======================================
    -- create_table(:tags)
       -> 0.0051s
    == 20180712114410 CreateTags: migrated (0.0053s) ==============================
    
    == 20180716103005 DeviseCreateUsers: migrating ================================
    -- create_table(:users)
       -> 0.0007s
    -- add_index(:users, :email, {:unique=>true})
       -> 0.0005s
    -- add_index(:users, :reset_password_token, {:unique=>true})
       -> 0.0007s
    == 20180716103005 DeviseCreateUsers: migrated (0.0025s) =======================
    
    == 20180716103123 AddNameToUsers: migrating ===================================
    -- add_column(:users, :nickname, :string, {:null=>false, :default=>""})
       -> 0.0007s
    -- add_column(:users, :major, :string, {:null=>false, :default=>""})
       -> 0.0003s
    -- add_column(:users, :another_major, :string)
       -> 0.0003s
    -- add_column(:users, :sex, :string)
       -> 0.0003s
    -- add_column(:users, :phone, :string, {:null=>false, :default=>""})
       -> 0.0003s
    -- add_index(:users, :nickname, {:unique=>true})
       -> 0.0010s
    -- add_index(:users, :phone, {:unique=>true})
       -> 0.0010s
    == 20180716103123 AddNameToUsers: migrated (0.0048s) ==========================
    
    == 20180718081457 CreateAdmissions: migrating =================================
    -- create_table(:admissions)
       -> 0.0019s
    == 20180718081457 CreateAdmissions: migrated (0.0020s) ========================
    
    == 20180718081707 CreateChats: migrating ======================================
    -- create_table(:chats)
       -> 0.0018s
    == 20180718081707 CreateChats: migrated (0.0019s) =============================
    
    == 20180720065211 CreateUserChatLogs: migrating ===============================
    -- create_table(:user_chat_logs)
       -> 0.0007s
    == 20180720065211 CreateUserChatLogs: migrated (0.0008s) ======================
    
    == 20180720065656 CreateReports: migrating ====================================
    -- create_table(:reports)
       -> 0.0007s
    == 20180720065656 CreateReports: migrated (0.0008s) ===========================
    

완료. 이제 secret key설정해주자!

우선 config폴더 밑에 application.yml이 있는지 확인

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ cd config/
    [ec2-user@ip-172-31-20-132 config]$ ls
    application.rb  database.yml    initializers  routes.rb
    boot.rb         environment.rb  locales       secrets.yml
    cable.yml       environments    puma.rb       spring.rb
    

여기에 application.yml이 없으니 프로젝트 폴더로 돌아와서

    [ec2-user@ip-172-31-20-132 config]$ cd ..
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ figaro install
          create  config/application.yml
          append  .gitignore
    

설치하고,  vi config/application.yml 을 console에 입력하여 편집해주자.

안에 내용은 기존에 application.yml에 넣은것 과 같지만, development: 대신 production: 을 써주고 그 아래에 붙여넣자. 

여기에 밑에 나온것을 복사하여

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ rake secret
    밑에 뭐 나옴 아무튼
    

vi config/application.yml 를 console에 치고 다음과 같이 설정

    ...
    SECRET_KEY_BASE: 복사한 것.
    

이제 migrate 다시 한번더 해준다.

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ RAILS_ENV=production rake db:migrate
    

이제 어떤 rake 명령에 있어 앞에 RAILS_ENV=production를 붙여준다.

** 오류가 뜹니다.(migrate가 안되는 syntax오류)

    [ec2-user@ip-172-31-20-132 config]$ ls
    application.rb   boot.rb    database.yml    environments  locales  routes.rb    spring.rb
    application.yml  cable.yml  environment.rb  initializers  puma.rb  secrets.yml
    [ec2-user@ip-172-31-20-132 config]$ cd environments/
    [ec2-user@ip-172-31-20-132 environments]$ ls
    development.rb  production.rb  test.rb
    [ec2-user@ip-172-31-20-132 environments]$ cd pro
    -bash: cd: pro: No such file or directory
    [ec2-user@ip-172-31-20-132 environments]$ vi production.rb 
    

이제 production.rb 수정을 해주도록한다. #을 붙여 임시로 주석처리해주고, 나중에 url붙이고 여기에 IP ADDRESS 대신에 넣어주도록 하자.

    ...
     # config.action_mailer.default_url_options = {host: IP Address, port: 443}
    ...
    

에러 수정하고 migrate하고 그다음에 이걸 입력해준다.

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ RAILS_ENV=production rake assets:precompile
    

자 여기서 오류나기 시작하면 노답입니다. 다행히 우리 조는 안났네 ㅎㅎㅎ

다시 서버를 틉시다.

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo /opt/nginx/sbin/nginx 
    

그러고 아까의 ip로 들어가면 사이트 접속이 잘됩니다.





도메인 달기!

aws 의  route53 으로 옵니다.

거기서 Hosted zones로 가서 다음과 같이 입력합니다.



하고 create 눌러줍시다. 그럼 이런 화면이 뜹니다.



create Record Set을 눌러준다. 한개는 아무것도 없이 value에 ip v4 주소만 넣어주고 create, 다른건 Name에 www를 넣어주고, value에 똑같이 넣어주고 create.



그럼 이런 상태



이제 해당 도메인 주소로 들어가면 들어갈 '준비'는 되어있는것.

이제 서버를 조정해주자

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo vi /opt/nginx/conf/nginx.conf
    

치고

    ...
    server {
            listen 80;
            passenger_enabled on;
            server_name muckbo.xyz // 임마 추가해준다.
            root /home/ec2-user/Muckbo_finish/public;
        }
    ...
    

하고 서버를 다시 껐다 킨다.

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo /opt/nginx/sbin/nginx -s stop
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ sudo /opt/nginx/sbin/nginx 
    

그 후 닷네임에서 관리가서 도메인 레코드 관리(DNS)로 가서 신청하기 누르고



aws route53에서 했던 것 처럼 A레코드에서 www없는거 하나, 있는거 하나 추가한다.

그 후 5분~하루(?) 정도 기다리고 저 도메인으로 접속하면 성공~~~



도메인도 잘 붙였으니, 이제 아까 Mailer의 host와 port를 고쳐봅시다.

 config/enviroment/production.rb

    ...
    # Do not dump schema after migrations.
      config.active_record.dump_schema_after_migration = false
    
      config.action_mailer.default_url_options = {host: "http://www.muckbo.xyz", port: 80}
    
    end
    

고치고, 서버를 다시 껐다 켜준다.

    [ec2-user@ip-172-31-20-132 environments]$ sudo /opt/nginx/sbin/nginx -s stop[ec2-user@ip-172-31-20-132 environments]$ sudo /opt/nginx/sbin/nginx 
    



주소를 https로 만들어보자

우선 서버를 꺼주고, su 권한으로가서 다음과 같이 입력해준다.

    [root@ip-172-31-20-132 ~]# curl -O https://dl.eff.org/certbot-auto
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100 62299  100 62299    0     0  62299      0  0:00:01 --:--:--  0:00:01  127k
    

그리고 다음과 같이 입력

    [root@ip-172-31-20-132 ~]# chmod +x certbot-auto
    

만든 녀석은 여기로 옮겨준다.

    [root@ip-172-31-20-132 ~]# mv certbot-auto /usr/bin/certbot-auto
    

옮긴뒤에 다음과 같이 명령어를 실행한다.

    [root@ip-172-31-20-132 ~]# certbot-auto certonly --standalone -d www.muckbo.xyz --debug
    

안된다... AMI 선택이 잘못되어서 ㅠ



수정하는 방법

1. 작업은 c9에서 합니다. 뭐 요거 저거 수정
2. 그리고 c9 터미널

    git status // 어떤 것이 수정되었는지 확인!
    git add . // 수정된 사항 전체 적용!
    git commit -m "커밋내용" // commit해주고
    git push // github에 upload
    

1. 배포서버 (ec2)에서 

    git pull
    

로 당깁니다.

1. 서버를 한번 껐다 켜줍니다.

    [ec2-user@ip-172-31-20-132 environments]$ sudo /opt/nginx/sbin/nginx -s stop[ec2-user@ip-172-31-20-132 environments]$ sudo /opt/nginx/sbin/nginx 
    

1. 혹시 css나 db를 수정했다면?

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ RAILS_ENV=production rake db:drop DISABLE_DATABASE_ENVIROMENT_CHECK=1
    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ RAILS_ENV=production rake db:migrate
    

이렇게 해줍니다.

1. console들어가고싶다면

    [ec2-user@ip-172-31-20-132 Muckbo_finish]$ RAILS_ENV=production rails c
    

라고 쳐줍니다.


본 내용은 <a href="https://github.com/namekun">김남혁의 github</a>에서 퍼온 것입니다.