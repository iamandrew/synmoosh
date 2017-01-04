---
title: synmoosh Development
layout: default
---


Vagrant
=======

synmoosh comes with vagrant setup, which will give you the following synmoosh development environment:
 
 * Ubuntu 16.04 with PHP 7
 * Apache configured to run as user vagrant (so no problems with the file permissions)
 * MySQL
 * Latest Moodle 3.1 installed
 * composer and synmoosh installed from source
 
Simply:

    % git clone https://github.com/tmuras/synmoosh
    % vagrant up
 
Your Moodle 3.1 is now available at http://192.168.33.10/moodle/ (login "admin", password "a").
 PhpMyAdmin URL is  http://192.168.33.10/phpmyadmin (MySQL login "root" and "mypassword).
 
Once you SSH into your box with:
 
     % vagrant ssh
     
You can run synmoosh inside the Moodle 3.1 installation:
 
     % cd /var/www/html/moodle
     % synmoosh user-list
     
The source code of synmoosh is under vagrant's home in /home/vagrant/synmoosh-src and calling "synmoosh" 
 command will actually call /home/vagrant/synmoosh-src/synmoosh.php.
  
The directory /home/vagrant/synmoosh-src is shared with your host machine as "synmoosh-src", in directory
 where you have cloned the git repository. This is so you can use your favourite IDE on your host PC.

So - no excuses now - use pre-configured environment, develop some awesome synmoosh commands and send 
 them to me! 

Performance information
=======================

You can use global option -t (or long version --performance) to show extra performance information collected while the command runs:

    % synmooshdev -t course-backup 2
    ... <output cut> ...
    *** PERFORMANCE INFORMATION ***
    Run from 2014-11-26T11:21:15+01:00 to 2014-11-26T11:21:16+01:00
    Real time run 0.667 seconds
    Server load before running the command: 0.14 0.16 0.16 1/584 6180
    Server load after: 0.14 0.16 0.16 1/584 6180
    Ticks: 67 user: 36 sys: 3 cuser: 0 csys: 0
    Memory use before command run (internal/real): 19058864/19136512 (18.18 MB/18.25 MB)
    Memory use after:  42252496/44040192 (40.30 MB/42.00 MB)
    Memory peak: 43679224/45088768  (41.66 MB/43.00 MB)
    *******************************



Functional tests
================

There are no unit tests implemented for testing synmoosh at the moment. Instead, a set of functional tests have been developed.
They are basically very simple bash scripts located in tests directory and named after command they test, e.g.:

    tests/file-list.sh

is used to test synmoosh file-list command.

All tests start with some common boilerplate:

    #!/bin/bash
    source functions.sh

    install_db
    install_data
    cd $MOODLEDIR

and then the test of the commmand is performed. Script should return (exit) with 0 if test is a success, with 1 otherwise. Here is test from file-list.sh:

    if synmoosh file-list id=6 | grep -w "grumpycat"; then
      exit 0
    else
      exit 1
    fi

All tests are then run with run-tests.php script, which in turn will generate status on the <a href="http://synmoosh-online.com/ci/">continues integration</a> page.


Environment
-----------

Notice in the test above that test suite assumes there is Moodle instance already setup and it contains a file called "grumpycat".
All commands will be run in a known, prepared environment with users, courses pre-created. “install moodle” means restoring Moodle DB and data from prepared snapshot.

Set up & run functional tests
--------------------------------

Some scripts use sudo chown command to operate on Moodle data, so to let them run without prompting for password add something like this to /etc/sudoers (use visudo to edit):

    %adm    ALL = NOPASSWD: /bin/chown, /bin/chmod

Then make sure your shell user is in group adm.

Create 2 directories for Moodle data, eg: ~/data/synmoosh-test/moodledata25 and ~/data/synmoosh-test/moodledata26. Give apache user write access to Moodle data dirs.

Create 2 empty databases: synmooshtest_25 and synmooshtest_26.

    #Get Moodle source code for 2.6 and 2.7
    cd ~/www/html/synmoosh/
    git clone https://github.com/moodle/moodle.git moodle25
    cd moodle25
    git checkout 3d176316cc1791e258a7c1b2118fd35976c9bcae
    cp config-dist.php config.php
    #configure settings in config.php down to & including $CFG->dataroot

    cd ~/www/html/synmoosh/
    cp -r moodle25 moodle26
    cd moodle26
    git checkout ba05f57
    cp config-dist.php config.php
    #configure settings in config.php down to & including $CFG->dataroot

    git clone https://github.com/tmuras/synmoosh
    cd synmoosh/tests
    #Configure DATA,DB,DBUSER and DBPASSWORD in restore_all.sh and run it
    ./restore_all.sh

Login to Moodle instances (e.g. http://localhost/synmoosh/moodle26/) as 'admin' using password 'a' and check if it works OK after restore.

    cp config-template.sh config25.sh
    cp config-template.sh config26.sh
    #configure variables in config25.sh and config26.sh

    #run tests, several should pass but some eventually fail:
    php run-tests.php


Contributing to synmoosh
=====================

1. Fork the project on github.
2. Follow "installation from Moodle git" section.
3. Look at existing plugins to see how they are done.
4. Create new plugin/update existing one. You can use synmoosh itself to generate a new command from a template for you:

    synmoosh generate-synmoosh category-command

5. Update this README.md file with the example on how to use your plugin.
6. For the extra bonus create a functional test to cover your command.
7. Send me pull request.


Local commands
==============

You can add your own, local commands to synmoosh by storing them in the same structure as synmoosh does but under ~/.synmoosh.
For example, to create your custom command dev-mytest that works with any Moodle version, you would put it under:

    ~/.synmoosh/Moosh/Command/Generic/Dev/MyTest.php
