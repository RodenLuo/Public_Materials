
This post tells how to install TPP v5.0.0 ([SourceForge](https://sourceforge.net/projects/sashimi/), [Wiki](http://tools.proteomecenter.org/wiki/index.php?title=Software:TPP)) on Ubuntu 16.04.1 LTS Linux. It follows the guide by the author, gets errors, and tells solutions. [Here](https://groups.google.com/forum/#!topic/spctools-discuss/oKBB1lSk894) is a short description of it. 

Caution: By following it, One should be able to install TPPv5 and use most (maybe all) of the function, but one will get an error when view the .pep.xml file by PepXML Viewer. I mentioned this error [here](https://groups.google.com/forum/#!topic/spctools-discuss/kS1bcTDWEJQ). Hope you can help.

###1. Get TPP v5 source file 
1. Download TPP_5.0.0-src.tgz from [here](https://sourceforge.net/projects/sashimi/files/Trans-Proteomic%20Pipeline%20%28TPP%29/TPP%20v5.0%20%28Typhoon%29%20rev%200/).
2. Unzip it by `tar -zxf TPP_5.0.0-src.tgz`
3. Move to the unzipped directory `cd TPP_5.0.0-src/`
Read `README`, read `BUILD_LINUX`. One can use `cat README` print the file.

###2. Follow the `Prerequisites` section in `BUILD_LINUX` and install the required software and libraries.
First, `sudo apt-get update`

1.  g++, Ubuntu should have it. `g++ --version` to check it. Mine is "g++ (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609"
2. Perl, should have it. `perl -v` to check. Mine is v5.22.1.
3. Apache, I will talk about it later.
4. gnuplot, `gnuplot --version` to check. Mine is "gnuplot 5.0 patchlevel 3". If you do not have it, you should be able to use `apt-cache madison gnuplot` (Courtesy of [lornix](http://superuser.com/questions/393681/how-to-find-out-which-versions-of-a-package-can-i-install-on-apt)) to see what version you can install and use `sudo apt-get install gnuplot` to install it.
5. xsltproc, the same as gnuplot.
6. C/C++ libraries, (I guess the `build` process has not made use of the already installed libraries but use the attached ones in the source file. Because I still got problems related to libraries even after I installed them. Anyway, install then first.)
  1. Use `apt-cache search libgd2` to search libraries. Then choose one output ending with "-dev", for me, I only have "libgd-dev" as output, to install.  As mentioned, use `apt-cache madison libgd-dev` to check package source if you want. `sudo apt-get install libgd-dev` to install it. 
  2. libpng, the same as libgd2. I chose libpng12-dev. (I do not know the differences among all the output in search. Seems libpng16-dev is the latest one, but latest sometimes encounter compatiblity problems, so I chose libpng12-dev. If not right, correct me. Thx!) `sudo apt-get install libpng12-dev` to install.
 3. The same for the other two, "zlib" and "libbz2". I use `sudo apt-get install zlib1g-dev` to install zlib, (Courtesy of [hwez](http://askubuntu.com/questions/508934/how-to-install-libpng-and-zlib/508937 )), and `sudo apt-get install libbz2-dev` to install libbz2.

###3. Configuring
Follow `BUILD_LINUX`. In dir `TPP_5.0.0-src`, create a file named `site.mk` and put your configuration info there. Command and content I use: 
command `vi site.mk`
content in "site.mk"

    INSTALL_DIR = /usr/tpp_install/tpp
    BASE_URL = /tpp
    TPP_DATA = /usr/tpp_install/tpp/data

###4. Building
In dir `TPP_5.0.0-src`, execute `sudo make all`.
Get error:
```
/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/extern/ProteoWizard/Makefile:353: warning: overriding recipe for target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/'
common.mk:448: warning: ignoring old recipe for target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/'
make: *** No rule to make target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/extern/ProteoWizard/pwiz-msi/', needed by '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/'.  Stop.
```
Find solution [here](https://groups.google.com/forum/#!topic/spctools-discuss/bBQLb0PLulg) (Courtesy of Jean-Christophe Ducom and Ali).
Here is a short description of that thread:
All command are supposed to be executed in dir `TPP_5.0.0-src`.

1. `mkdir -p ./extern/ProteoWizard/pwiz-msi`
2. `make pwiz-msi` (Get error for sure, ignore it.)
3. `vi ./extern/ProteoWizard/pwiz-msi/VERSION` Replace the space with dot. Here is the file after editing.

        3.0.10185
4.  `vi ./extern/ProteoWizard/Makefile` Comment out the following lines (Line number: 466, 467). In the file part below, the two lines are already commented out.

        # rm -f $(PWIZ_MSIDIR)/VERSION
        # wget -nv -O $(PWIZ_MSIDIR)/VERSION $(PWIZ_TCMSI)/VERSION\?guest=1
5. `make pwiz-msi` Succeed.
6. `sudo make all` I still got problems. Here is the last three lines of the output.

        ...failed updating 1 target...
        extern/Makefile:262: recipe for target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/lib/libboost_system.a' failed
        make: *** [/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/lib/libboost_system.a] Error 1
7. When I  `sudo make all` again, the error changed to:

        ...failed updating 1 target...
        extern/Makefile:262: recipe for target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/lib/libboost_thread.a' failed
        make: *** [/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/lib/libboost_thread.a] Error 1

###5. Replace the boost lib
Regarding the above error, I found a [hint](https://groups.google.com/forum/#!searchin/spctools-discuss/has_icu$20builds$20$20$20$20$20$20$20$20$20$20$20$3A$20no$20%7Csort:relevance/spctools-discuss/iBLsDJLy5Vo/T_dxXZAISYcJ) (Courtesy of Jagan Kommineni  and bmcnally) and fixed it in a different manner. I actually followed the solution first, installed the newest [boost](http://www.boost.org/users/history/version_1_62_0.html) libraries. But it seems that the `make` logic has been changed in TPP5.0.0. I do not know where to indicate making process to use the boost I installed like Jagan used in the hint link. I directly copied the libboost_thread.a from the boost lib I installed to dir `TPP_5.0.0-src/build/gnu-x86_64/lib/`, I could  get rid of the above error. But there were other errors coming later. After two more errors (I will mention it in another post.) by solving it this way (copying the required files), I started trying the another way in my mind, replacing the boost library source file in TPP\_5.0.0-src. Here are the steps.

All command are supposed to be executed in dir `TPP_5.0.0-src`.

1. Remove the old boost source files in extern directory. `rm extern/boost_1_54_0.*`
2. Download [boost\_1\_62\_0.tar.bz2](http://www.boost.org/users/history/version_1_62_0.html) and move it to dir `TPP_5.0.0-src/extern`. Change the right on this file to the rights the same as other .bz2 under that dir. `chmod 664 extern/boost_1_62_0.tar.bz2`.
3. Edit `TPP_5.0.0-src/extern/Makefile` to change the version and remove the parts related to the patches for the old version. `vi extern/Makefile` Here is the file part after editing.

        # line 225, change the number
        BOOST_VER := 1_62_0
        ...
        # line 228, comment it out
        # BOOST_PATCHES := $(wildcard $(TPP_EXT)/boost_$(BOOST_VER)*patch)
        ...
        # line 245,248, comment them out
        # $(foreach P, $(BOOST_PATCHES),                                        \
        #    @echo "### ...applying Boost patch $(notdir $(P))" $(nl)   \
        #    patch -s -p0 -d $(dir $(BOOST_SRC)) < $(P) $(nl)           \
        # )

4. `sudo make all` GOES LIKE A MAGIC. (Takes about 20min)

###4. Installing
All command are supposed to be executed in dir `TPP_5.0.0-src`.

1. `sudo make install` gets error:

        checking libpng-config script... configure: error: png support requested, but not found at requested location: /home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/bin/libpng-config
        extern/Makefile:849: recipe for target '/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/artifacts/libgd-2.1.0/' failed
        make: *** [/home/roden/proteomics/bin/TPP_install/TPP_5.0.0-src/build/gnu-x86_64/artifacts/libgd-2.1.0/] Error 1

2. I have libpng-config 1.2.54 on the Linux (It is installed when I performed the prerequisites checking. Obviously, the `make install` has not used the installed libpng). Feel there may still be problem if I use it. So I manually installed the extern/libpng-1.5.19.tar.gz to /usr/local/bin/libpng-config. Then edit the configure file in libgd-2.1.0.
  1. `tar -zxf extern/libpng-1.5.19.tar.gz`
  2. `cd libpng-1.5.19`, Follow the guide in `cat INSTALL`
  3. `./configure`
  4. `make check`
  5. `sudo make install`
  6. `cd ../`
  7. `sudo rm -r libpng-1.5.19/`
  8. `vi build/gnu-x86_64/artifacts/libgd-2.1.0/configure` Edit

              # line 12703, comment it out and add a new line
              # LIBPNG_CONFIG=$with_png/bin/libpng-config
              LIBPNG_CONFIG=/usr/local/bin/libpng-config
  9. `sudo make install` goes smoothly.

###5. Test the tools
The reason I need to install TPP is that I need to run `xinteract -OA -PPM -Nbasename basename.pep.xml`. Now that I have tpp installed. So I test it. I get this error.

```
Can't locate FindBin/libs.pm in @INC (you may need to install the FindBin::libs module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.22.1 /usr/local/share/perl/5.22.1 /usr/lib/x86_64-linux-gnu/perl5/5.22 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.22 /usr/share/perl/5.22 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/tpp_install/tpp/bin/ProphetModels.pl line 33.
BEGIN failed--compilation aborted at /usr/tpp_install/tpp/bin/ProphetModels.pl line 33.
``` 

Follow [this](http://www.cpan.org/modules/INSTALL.html) (Courtesy of CPAN) to install FindBin::libs by: `cpan App::cpanminus`, and then `cpanm FindBin::libs`.

Then I am able to see "job completed in 26 sec". May be there are other compatibility issues. But for now, the command line functions I need run well. Then let's take a look at the web GUI.

###6. Configure Web GUI
1. Install apache2. `sudo apt-get install apache2`. The version I got installed is "2.4.18-2ubuntu3.1". Follow [this](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04). (Courtesy of Brennen Bearnes) Remember add one line "ServerName server\_domain\_or\_IP" to `/etc/apache2/apache2.conf` file to suppress a warning message as the link mentioned. 
2. Add the `httpd-tpp.conf` to the right place
  1. `cd /etc/apache2/sites-available`
  2. `sudo cp /usr/tpp_install/tpp/conf/httpd-tpp.conf .`
  3. `sudo chmod 644 httpd-tpp.conf` for the convenience of  later editing.
  4. `cd ../sites-enabled`
  5. `sudo rm 000-default.conf`
  6. `sudo ln -s ../sites-available/httpd-tpp.conf .`
  7. `cd ../sites-available`
  8. `sudo vi httpd-tpp.conf` The file part after editing. The line showing "# Line" does _NOT_ exist in the file.

            # Line 26
            Listen 80 # You can change to the port you want, but do NOT forget to allow the traffic for that port, like changing the configuration of ufw
  9. `sudo apache2ctl configtest` get error:
            
            AH00526: Syntax error on line 52 of /etc/apache2/sites-enabled/httpd-tpp.conf:
            Invalid command 'ScriptInterpreterSource', perhaps misspelled or defined by a module not included in the server configuration
            Action 'configtest' failed.
            The Apache error log may have more information.
  10. Found [this](http://stackoverflow.com/questions/8680615/ignore-apache-directive-scriptinterpretersource-on-linux) (Courtesy of Graham Dumpleton). But I directly commented it out. `sudo vi httpd-tpp.conf`

            # line 52, comment it out
            # ScriptInterpreterSource Registry

  11. `sudo apache2ctl configtest` get error:

            [Tue Nov 15 19:30:27.466357 2016] [env:warn] [pid 1400:tid 140719947372416] AH01506: PassEnv variable TPP_HOME was undefined
            [Tue Nov 15 19:30:27.466462 2016] [env:warn] [pid 1400:tid 140719947372416] AH01506: PassEnv variable TPP_DATADIR was undefined
            AH00526: Syntax error on line 113 of /etc/apache2/sites-enabled/httpd-tpp.conf:
            Invalid command 'RewriteEngine', perhaps misspelled or defined by a module not included in the server configuration
            Action 'configtest' failed.
            The Apache error log may have more information.
  12. Comment out the two "PassEnv" and Make the "SetEnv" into effect. `sudo vi httpd-tpp.conf`

            # line 55-63
            #    PassEnv TPP_HOME
            #    PassEnv TPP_DATADIR
            #   PassEnv TPP_BASEURL
            #   PassEnv TPP_DATAURL
                # ...or override TPP's enviroment variables
               SetEnv TPP_HOME    /usr/tpp_install/tpp
               SetEnv TPP_DATADIR /usr/tpp_install/tpp/data
               SetEnv TPP_BASEURL tpp
               SetEnv TPP_DATAURL tpp/data

  13. Follow [this](http://askubuntu.com/questions/64454/how-do-i-enable-the-mod-rewrite-apache-module-for-ubuntu-11-04) (Courtesy of zoopster). `sudo a2enmod rewrite`, `sudo apache2ctl configtest` check should output "Syntax OK", `sudo service apache2 restart`. Check the webpage by going to `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl`. Get error:

            Forbidden

            You don't have permission to access /tpp/cgi-bin/tpp_gui.pl on this server.
            Apache/2.4.18 (Ubuntu) Server at YOUr_Server_IP Port 80

  14. Add log file path to conf file. `sudo vi httpd-tpp.conf` add these two lines in the bottom of the file.

            CustomLog /var/log/apache2/tpp_access.log common
            ErrorLog /var/log/apache2/tpp_error.log
  15. Restart apache2 by `sudo service apache2 restart`.  Change to root user `sudo su root`. Move to log dir `cd /var/log/apache2`. Visit `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl` again. `cat tpp_error.log`

            [Tue Nov 15 20:49:30.276029 2016] [access_compat:error] [pid 1996:tid 140129635923712] [client 10.0.2.2:55543] AH01797: client denied by server configuration: /usr/tpp_install/tpp/cgi-bin/tpp_gui.pl
  16. Follow [this](http://stackoverflow.com/questions/18392741/apache2-ah01630-client-denied-by-server-configuration) (Courtesy of Jayakumar Bellie). `sudo vi httpd-tpp.conf`

            # line 32-43, Add a new line "Require all denied", comment out the rest.
            <Directory "/usr/tpp_install/tpp">
                AllowOverride None
                Require all granted
                # Allow only access from localhost:
                # Order deny,allow
                # Deny from all
                # Allow from 127.0.0.0/255.0.0.0 ::1/128
                # Or use the following to allow everyone from anywhere access:
            #   Order allow,deny
            #   Allow from all
            </Directory>

  17. `sudo service apache2 restart`. Visit `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl` again. Get the plain text of "tpp_gui.pl" in browser. The first line is:

            #!/usr/bin/perl

  18. Insight from [this](http://stackoverflow.com/questions/14792978/perl-apache-perl-script-displayed-as-plain-text) (Courtesy of user966588). `sudo a2enmod cgi`, and `sudo service apache2 restart`. Visit `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl` again. Get something, but not perfect. Use guest; guest to login, it goes to `http://YOUr_Server_IP/tpp/cgi-bin/tpp/cgi-bin/tpp_gui.pl`
![](https://github.com/RodenLuo/Public_Materials/blob/master/image/QQ20161115-0.png)

###7. Configure Web GUI Cont'd (Path issues)
Check the error above, We can know that it is because wrongly used path. It recognized as relative path, but it should be absolute path. Below is the solution. (My debugging route. At first I thought this is the problem of the configuration of apache2 because I knew that on Windows it works well. I tried to find ways to tell apaches2 should use absolute path but failed. Then I checked file "tpp\_gui.pl", looked up "images/isb\_logo.png", then looked up "tpp\_html\_url", then went to "tpp/lib/tpplib\_perl.pm", looked up "getHtmlUrl", then "\_lookup", then I noticed "$ENV{TPP\_HOME}". Then I realized SetEnv stuffs in httpd-tpp.conf file. Then somehow I noticed that I made "SetEnv TPP\_BASEURL tpp" and "#SetEnv TPP\_DATAURL tpp/data" in effect in step 6.12 and thought it should be "/tpp" and "/tpp/data", so I added the forward slash, "/". It turned out working. But when I commented them out, it also worked. So here I use the setting come with the TPP 5.0.0 release, comment out the two lines. This took me about 6 hours :( to find it.)

1. `sudo vi httpd-tpp.conf`, comment out two lines.

        # line 62,63 comment them out
        # SetEnv TPP_BASEURL tpp
        # SetEnv TPP_DATAURL tpp/data
2. Visit `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl`. Then it seems going well.
![](https://github.com/RodenLuo/Public_Materials/blob/master/image/QQ20161115-1.png)

But when I loged into it. It showed as:
![](https://github.com/RodenLuo/Public_Materials/blob/master/image/QQ20161115-2.png)

Error msg:

    CANNOT_CREATE_SESSION_FILE:/usr/tpp_install/tpp/users/guest/session_ADUDH5YTX:Permission denied: A fatal error has been encountered.
    NOLOGFILE: Cannot open /usr/tpp_install/tpp/log/tpp_web.log for writing: Permission denied 

###8. Configure Web GUI Cont'd (Rights issues and others)
1. check log `sudo cat /var/log/apache2/tpp_error.log`. It says:

        touch: cannot touch '/usr/tpp_install/tpp/log/tpp_web.log': Permission denied
2. `cd /usr/tpp_install`, change the onwership of the dir by `sudo chown -R www-data:www-data tpp`.
3. Visit `http://YOUr_Server_IP/tpp/cgi-bin/tpp_gui.pl`. Everything seems going well now. Upload a pep.xml file to view. The viewer works fin. But I notice this error in the head of the web page:
 
        Messages    
        Unable to access xml file via webserver, so cannot check xml file for changes ()
        Check webserver configuration if desired.
        (URL: undefined)
4. Check log `sudo cat /var/log/apache2/tpp_error.log` Notice error:

        Can't locate FindBin/libs.pm in @INC (you may need to install the FindBin::libs module) (@INC contains: /usr/tpp_install/tpp/lib/perl /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.22.1 /usr/local/share/perl/5.22.1 /usr/lib/x86_64-linux-gnu/perl5/5.22 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.22 /usr/share/perl/5.22 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/tpp_install/tpp/bin/tpp_models.pl line 34.
        BEGIN failed--compilation aborted at /usr/tpp_install/tpp/bin/tpp_models.pl line 34.

5. This is rather weird, because I have installed FindBin::libs it in step 5. I guess may be resulted from some version compatibility issues, or the perl apache used is not the same as I used in step 5. Anyway, I create a short link for it.
  1. `cd /usr/share/perl/5.22`
  2. `sudo mkdir FindBin`, `cd FindBin`
  3. `sudo ln -s ../FindBin.pm libs.pm`

6. Try the pep.xml viewer in the web GUI. The Error/Warning msg still shows up. But there is no more error in apache2 log file.

7. I have checked the source code in `/usr/tpp_install/tpp/html/PepXMLViewer.html`, and added code to check "dateorig" and "lastupdate", they are both "undefined". Since it works well. I'm gonna take it for now. I mentioned this error [here](https://groups.google.com/forum/#!topic/spctools-discuss/kS1bcTDWEJQ).Hope you can help. Thx!
