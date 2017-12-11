# Beyond_Netcool_WebGUI_Tools
The basic tool types don't always get you where you need to go.  Sometimes you need form validation, or you want to pre-populate form items programatically.  Maybe you want to run selects against ObjectServer tables.  Use some CGI, someHTML, and some REST calls to the ObjectServer to get the job done.  This is a simple example thatsolves a real problem for my customer.

**HTML forms need a place to live**

If you are using the DASH/Jazz for Service Management web server that comes with WebSphere Application Server you can add HTML files to the "myBox" directory. Here is a quick lesson:

 - Become the user that owns the Jazz files (netcool in my case)

        su - netcool

 - CD into the myBox dir

        cd ... wherever.../JazzSM/ui/myBox/

 - Edit your html files in myBox/web_files/

 - Deploy your files Note: enclose the smadmin password in doublequotes and escape those special chars

        ./deployMyBox.sh -username smadmin -password "C\$tfood"

 - This is then the URL: https://1.2.3.4:16311/myBox/yourfile.html

 - For example, my filename is initiateSWAT.html, so: https://1.2.3.4:16311/myBox/initiateSWAT.html
