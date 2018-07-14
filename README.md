# Steps to setup MapProxy on IIS 7.5+ using python 2.7 and WfastCGI
(this has only been tested on windows server 2016 but theres no reason it shouldnt work on 7, 8, 10, 2008 & 2012)

1. install CGI in IIS (https://docs.microsoft.com/en-us/iis/configuration/system.webserver/cgi)
2. install python 2.7.* for Windows (https://www.python.org/downloads/). Making sure to enable the 'Set Path' option in the installer and install for all users. I would recommend installing to 'c:\python27'.
3. open an elevated powershell window. (Command prompt will work fine aswell without the './' in the commands below)
4. type 'pip install virtualenv' + enter. this installs the python virtual environment module.
5. create a virtual environment for mapproxy by typing 'virtualenv [TARGETDIR]' + enter. where TARGETDIR is the directory youd like to install to. example: virtualenv c:\inetpub\mapproxy
6. type 'cd [TARGETDIR]\scripts' + enter
7. type './activate' + enter. powershell should now have the virtualenv name appended to the prompt e.g.
```
(mapproxy) PS C:\inetpub\[TARGETDIR]\Scripts>
```
### YOU MUST COMPLETE STEPS 6 & 7 EVERYTIME YOU OPEN A NEW POWERSHELL WINDOW TO WORK IN THE VIRTUALENV
8. type 'pip install mapproxy' + enter
9. download and install the Microsoft Visual C++ Compiler for Python 2.7 from http://aka.ms/vcpython27
10. type 'pip install pyproj' + enter
11. type 'pip install wfastcgi' + enter
12. to test mapproxy is working type 'mapproxy-util --version' + enter. should display version number like 'MapProxy 1.11.0'
13. type 'mapproxy-util create -t base-config c:\inetpub\\[TARGETDIR]\\[APPNAME]' + enter. where [APPNAME] is the name of your appilcation. This creates the mapproxy demo yaml files.
14. type 'mapproxy-util create -t wsgi-app -f c:\inetpub\\[TARGETDIR]\\[APPNAME]\mapproxy.yaml c:\inetpub\[TARGETDIR]\[APPNAME]\config.py'. where [APPNAME] is the name of your appilcation from the previous step. this will create a simple WSGI app config file.
15. type 'mapproxy-util create -t log-ini c:\inetpub\[TARGETDIR]\[APPNAME]\log.ini'. This creates the logging config file.
16. open 'c:\inetpub\\[TARGETDIR]\\[APPNAME]\config.py' with notepad and change 'application = make_wsgi_app(...' to 'app = make_wsgi_app(...' and uncomment the lines below. 
### MAKE SURE THERE ARE ALL BACKSLASHES (\\) IN THE LOGFILE PATH.
```python
# from logging.config import fileConfig
# import os.path
# fileConfig(r'c:\inetpub\[TARGETDIR]\[APPNAME]/log.ini', {'here': os.path.dirname(__file__)})
```
config.py should read something like this
```python
from logging.config import fileConfig
import os.path
fileConfig(r'c:\inetpub\[TARGETDIR]\[APPNAME]\log.ini', {'here': os.path.dirname(__file__)})

from mapproxy.wsgiapp import make_wsgi_app
app = make_wsgi_app(r'c:\inetpub\[TARGETDIR]\[APPNAME]\mapproxy.yaml')
```
17. open IIS and goto 'FastCGI Settings' for the server. Add an application with:
```
FULL PATH=C:\inetpub\[TARGETDIR]\Scripts\python.exe
ARGUMENTS=C:\inetpub\[TARGETDIR]\Lib\site-packages\wfastcgi.py
```
18. once saved 'Edit...' the new 'FastCGI' entry and add the following 'Environmental Variables'
```
PYTHONPATH=C:\inetpub\[TARGETDIR]\Scripts\
WSGI_HANDLER=config.app
```
18. in IIS manager add a new site with any name and 'c:\inetpub\\[TARGETDIR]\\[APPNAME]' as the physical path (https://docs.microsoft.com/en-au/iis/get-started/getting-started-with-iis/create-a-web-site)
19. in the new site settings goto 'Handler Mappings' and 'Add Module Mapping...' using settings below
```
Request Path = *
Module = FastCgiModule
Executable Path = c:\inetpub\[TARGETDIR]\scripts\python.exe|c:\inetpub\[TARGETDIR]\lib\site-packages\wfastcgi.py
Name = WSGI
Request Restrictions > 'Invoke handler only if request is mapped to' = unchecked
```
20. change file permissions for 'C:\inetpub\\[TARGETDIR]\' to give 'Full Control' to user group 'IIS_USRS'. Including child objects.
21. right click on the site in IIS and click 'Manage Site > Browse' and you should see the mapproxy demo site. These settings can be viewed and changed in the mapproxy.yaml file.
## Donesky
