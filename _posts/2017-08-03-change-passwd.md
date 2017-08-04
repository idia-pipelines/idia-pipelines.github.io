---
layout: post
title: "Change your Password!"
date: 2017-08-04 10:02:00 +02000
---

Hi Folks, 

If you've received an account you're probably wondering on how to change your password. You can do
this as follows (in one line) 

`$ ldappasswd -H ldap://10.102.4.109 -x -D "cn=username,ou=users,dc=idia,dc=arc,dc=ac,dc=za" -W -S -A`

*Important:* In the command above, you will need to change `username` to your _username_.

You will then be prompted for passwords as follows:
* Twice for your current password (verification).
* Twice for your new password. 
* For your _LDAP_ password, which will be your *old* password. This is required to bind to the
  LDAP server to commit the change to your password. 

This is a little more involved then usual, because your credentials are not specific to a machine.
Access on the cloud is coordinated on using the LDAP server -- this is why you need to do the
_authenticate -> enter new password -> authorize/bind_ process to propagate your password to the
server. This allows us to provide you with access to any machine provisioned for your project
without having to remember a myriad of passwords. 

Please contact us if you have any questions. This information is also available in our
[Access][access] page.

[access]: /access/
