LDAP SSO
========

The LDAP Single Sign-On module implements Single Sign On (SSO) LDAP
Authentication, provides an administrator with the ability to configure
a Backdrop site to use either NTLM or Kerberos to authenticate Backdrop users.

In short, it relies on the $_SERVER web server variable to integrate a site
within a managed domain. The net effect is that either automatically, or by
visiting a link, a user is authenticated and logged into a Backdrop site without
requiring the user to manually enter credentials on suitably configured
installations.

Note that this module is not an SSO provider usable over the public internet
without additional middleware. Have a look at SimpleSAML and similar modules
if you are looking for this.


Dependencies
------------

Use of the module requires that you download and install two submodules of LDAP
module:
- LDAP Servers
- LDAP Authentication

LDAP module: https://backdropcms.org/project/ldap

Note: The Simple LDAP module also includes an implementation of SSO:
https://backdropcms.org/project/simple_ldap


Installation
------------

- Install this module using the official Backdrop CMS instructions at
  https://backdropcms.org/guide/modules.
- Configuration page: 
  - admin/config/people/ldap/authentication
  - Administration > Configuration > User accounts > LDAP Configuration > Authentication > Single Sign-on
- Check the log later. If you see several errors with this message, disable the "Cache pages for anonymous users" option:
  "Warning: Cannot modify header information - headers already sent in ldap_sso_boot()"
  Administration > Configuration > Development > Performance > Disable: Cache pages for anonymous users

	
Usage instructions
------------------

To use the single sign-on feature, your web server must provide an authentication
mechanism for LDAP. The only authentication mechanism used in development
was mod_auth_sspi for Apache/Windows, but so long as the web server's LDAP
authentication mechanism is configured to provide the $_SERVER variable
$_SERVER['REMOTE_USER'] or $_SERVER['REDIRECT_REMOTE_USER'] corresponding
directly to a user's LDAP user name, this should work all the same. This
will require some sort of LDAP authentication mechanism; mod_auth_sspi is
available here: https://sourceforge.net/projects/mod-auth-sspi/,
while mod_ntlm is available here: http://modntlm.sourceforge.net/,
and mod_auth_ntlm_winbind is available here:
https://www.samba.org/ftp/unpacked/lorikeet/mod_auth_ntlm_winbind/
If a Linux distribution is being used, Apache authentication modules are likely
available within the distro's package manager.

Unless an administrator wishes to require that all visitors be authenticated,
NTLM and/or basic authentication should be set up only on the path
user/login/sso, which will authentify the visitor but not deny access to view
the site if the visitor is not authenticated. An administrator may wish to
require LDAP authentication to view any portion of the site; this can be
achieved by changing the location directive below to "/". An administrator may
also wish to automatically log in visitors to Backdrop; this can be achieved by
checking "Turn on automated single sign-on" in the modules' configuration page.

An example of an Apache configuration for a named virtualhost configuration
using mod_auth_sspi on Windows is as follows:


httpd.conf:
```
# Virtual hosts
Include conf/extra/httpd-vhosts.conf

# Pass NTLM authentication to Apache
LoadModule sspi_auth_module modules/mod_auth_sspi.so

<IfModule !mod_auth_sspi.c>
  LoadModule sspi_auth_module modules/mod_auth_sspi.so
</IfModule>
```


httpd-vhosts.conf:
```
NameVirtualHost example.com

<VirtualHost example.com>
  DocumentRoot "D:/www/example.com/htdocs"
  ServerName example.com

  <directory "D:/www/example.com/htdocs">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order Allow,Deny
    Allow from all
  </directory>

  <Location /user/login/sso>
    AuthType SSPI
    AuthName "Example.com - Login using your LDAP user name and password"
    SSPIAuth On
    SSPIAuthoritative On
    ### The domain used to authenticate with LDAP; this should match the domain
    ### configured in the LDAP integration configuration within Backdrop
    SSPIDomain ad.example.com
    SSPIOmitDomain On
    SSPIOfferBasic On
    Require valid-user
    #SSPIBasicPreferred On
    #SSPIofferSSPI off
  </Location>
</VirtualHost>
```

After enabling and configuring an LDAP authentication module within Apache,
visit user/login/sso in the Backdrop installation on example.com. With or without
the ldap sso feature enabled, the browser should prompt for a user name and
password if using Internet Explorer 8 or a non-Microsoft browser. Internet
Explorer 7 by default will pass NTLM authentication credentials to local
websites, and IE8 and Firefox can be configured to do this as well.

If prompted for credentials on that path, enter a valid LDAP user name,
omitting the domain if "SSPIOmitDomain On" is configured, as well as a password.
If the credentials are correct, or if NTLM credentials are passed automatically
by the browser and successfully authenticated, a Backdrop 404 "Page not found"
message will be displayed if the module is not enabled; an "access is denied"
message will be displayed if the module is enabled and the browser is already
logged in; and if the ldap_sso module is fully configured and there is no
existing session, the browser will display the message "You have been
successfully authenticated" after redirecting to the sites' home page if you
have checked "Notify user of successful authentication".



Issues
------

Bugs and Feature requests should be reported in the Issue Queue:
https://github.com/backdrop-contrib/ldap_sso/issues


Current Maintainer
------------------

- Attila Vasas (https://github.com/vasasa).
- Seeking additional maintainers.


Credits
-------

- Ported to Backdrop CMS by Attila Vasas (https://github.com/vasasa).
- Originally written for Drupal by Hendrik Grahl (https://www.drupal.org/u/grahl).


License
-------

This project is GPL v2 software. See the LICENSE.txt file in this directory for
complete text.


Screenshot
----------

![configuration](https://github.com/backdrop-contrib/ldap_sso/blob/1.x-2.x/images/screenshot.png)
