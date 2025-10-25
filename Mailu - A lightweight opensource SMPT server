I want to build a SMPT server on my vps. Why? I need to use it for my Vaultwarden server to send verification emails. I chose Mailu as it's lightweighted and simple to build. 

I'll show you the minimum steps for a working mail server
## 1. Generate docker compose file

One thing I like mailu is that it provides a setting file wizard for you so you can easily configure those adjustment and settings on it.

>[mailu setup utility](https://setup.mailu.io/) 

here are essential configurations

1. **desired domain name** in ``Main mail domain and server display name.``

![[Pasted image 20251014012516.png]]

2. Choose a web email client
![[Pasted image 20251014013356.png]]

3. Put your server's public ip address in `IPv4 listen address` 
![[Pasted image 20251014013439.png]]

4. and make sure there's no conflict. you can use `ip a` or `docker network ls` `docker network inspect <network_name>`
![[Pasted image 20251014013549.png]]

5. a sub domain name for web admin UI or client UI
![[Pasted image 20251014013707.png]]

Finally the wizard will generate wget and docker commands for you to compose. Easy does it. 



When your docker is up and running, run the last command they provide to you. This will create an admin account to manage following settings.

 
## 2.  DNS setting

In the last step of section one, we specified a web domain name for our mail service. Let map it to server's IP.

![[Pasted image 20251014014622.png]]

You'll need to wait for 10 minuites for DNS to take effect. if you have other web services running on the same machine, it'd be better to use [[Reverse Proxy]] .

Visit mai.yourdomain.com. Login to admin with the password. Click Sign in Admin to enter admin dashboard.
![[Pasted image 20251014015645.png]]

Click Mail Domains in the sidebar
![[Pasted image 20251014015944.png]]


Click "details" under Actions column. Create new domain if it's not appeared in the list.
![[Pasted image 20251014020320.png]]

generate keys or any other domain records you wish to add. Then you can either copy past or download zone file to port to the DNS. This will take few minuintes.
![[Pasted image 20251014020610.png]]

## 3. Add user/ using third-party client.

before we add user, we can go to Webmail in the sidebar sending an email with admin account to test the reachability.
![[Pasted image 20251014021738.png]]

Under Domain List as previous step, Click Users under Manage column. then you will see New user bottom at right-top corner.
![[Pasted image 20251014022036.png]]

You can use third-party client using these information listed in Client setup. 
![[Pasted image 20251014022404.png]]

## 4. test and improvement

I sent a test message on https://www.mail-tester.com/ 

Here's the result:
![[Pasted image 20251014022933.png]]

I'm so broken so can only afford a .top domain
![[Pasted image 20251014023248.png]]

no rDNS. Let's setup one
![[Pasted image 20251014023335.png]]

Typically, a reverse DNS (PTR) record must be configured with the provider that manages the IP address — in this case, my VPS provider.

They have a pretty straight forward interface. Just type in your mail. subdomain and ip address.
![[Pasted image 20251014125037.png]]

Let's test again. We got a much higher point.
![[Pasted image 20251014125156.png]]

Test it with my outlook inbox. it was still identified as a junk. maybe because of the .top domain.

![[Pasted image 20251014125455.png]]

