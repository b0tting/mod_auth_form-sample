mod_auth_form sample
==========
A sample page for how to use mod_auth_form in Apache 2.4+ under Ubuntu / Debian.
It uses some javascript to allow parametrization and detect if a login has failed to display a friendly message to the user.

### Installation 
1. Enable the required mods in Apache

        sudo a2enmod session
        sudo a2enmod session_cookie
        sudo a2enmod auth_form
        sudo a2enmod request

2. Create a folder, preferably under /var/www and copy index.html and css to that folder
3. Create a password-file with ```htpasswd -c /etc/apache2/.htpasswd testuser```
4. Modify the Directory-tag for the DocumentRoot to contain the follow in your apache-config (```/etc/apache2/sites-enabled/000-default.conf```) 

        <Directory /var/www/>
                AuthFormProvider file
                AuthName "authenticationform"
                AuthType form
                AuthUserFile /etc/apache2/.htpasswd
                AuthFormLoginRequiredLocation "http://www.myformlocation.com/index.html?target=http://www.mylocation.com&title=Please login with your credentials"
                Session On
                SessionCookieName loginsession path=/
        </Directory>
 
5. Add a tag for each URL that needs to be protected by the login to the same file

        <Directory /var/www/protected_folder>
                Require valid-user
        </Directory>

6. Restart Apache with ```sudo /etc/init.d/apache2 restart```

### A complicated example 
Ths example uses a centralized login form and redirects users to your given location after entering their credentials:

    <VirtualHost *:80>
     ServerName www.mylocation.com


     LogFormat "%h %l %u %t \"%r\" %>s %b" common
     CustomLog  ${APACHE_LOG_DIR}/mylocation_access_log.log common
     <Location />
        Order deny,allow
        Deny from all
        ## allow some addresses to pass through without password challenge
        Allow from 192.168.1.     
        AuthFormProvider file
        AuthName "authenticationform"
        AuthType form
        AuthFormLoginRequiredLocation "http://www.myloginformlocation.nl/index.html?target=http://www.mylocation.com&title=Username and password?"
        Session On
        SessionCookieName mysession path=/
        AuthUserFile /etc/apache2/.htpasswd
        
        Require valid-user
        Satisfy Any
        
        ProxyPass  http://192.168.1.2:8080/
        ProxyPassReverse  http://192.168.1.2:8080
     </Location>
    </VirtualHost>
~
~



