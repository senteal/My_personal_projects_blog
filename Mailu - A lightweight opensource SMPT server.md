I want to build a SMPT server on my vps. Why? I need to use it for my Vaultwarden server to send verification emails. I chose Mailu as it's lightweighted and simple to build. 

I'll show you the minimum steps for a working mail server
## 1. Generate docker compose file

One thing I like mailu is that it provides a setting file wizard for you so you can easily configure those adjustment and settings on it.

>[mailu setup utility](https://setup.mailu.io/) 

here are essential configurations

1. **desired domain name** in ``Main mail domain and server display name.``

<img width="406" height="92" alt="Pasted image 20251014012516" src="https://github.com/user-attachments/assets/e8992a68-3739-451c-9c1d-ceac57c1764d" />

2. Choose a web email client
<img width="312" height="339" alt="Pasted image 20251014013356" src="https://github.com/user-attachments/assets/86e8eebe-de8c-4765-8dac-b677fcf3c4a5" />

3. Put your server's public ip address in `IPv4 listen address` 
<img width="201" height="83" alt="Pasted image 20251014013439" src="https://github.com/user-attachments/assets/7d7eb5c0-8b71-4cb7-b851-1801c0363284" />

4. and make sure there's no conflict. you can use `ip a` or `docker network ls` `docker network inspect <network_name>`
<img width="1690" height="101" alt="Pasted image 20251014013549" src="https://github.com/user-attachments/assets/be551b27-6f7b-41ab-97e4-6322b20517d0" />

5. a sub domain name for web admin UI or client UI
<img width="345" height="103" alt="Pasted image 20251014013707" src="https://github.com/user-attachments/assets/e65cef5e-ba4b-4811-838e-37045f6c587c" />

Finally the wizard will generate wget and docker commands for you to compose. Easy does it. 



When your docker is up and running, run the last command they provide to you. This will create an admin account to manage following settings.

 
## 2.  DNS setting

In the last step of section one, we specified a web domain name for our mail service. Let map it to server's IP.
<img width="1049" height="79" alt="Pasted image 20251014014622" src="https://github.com/user-attachments/assets/67e58411-4696-4b62-b329-94dc084a478b" />


You'll need to wait for 10 minuites for DNS to take effect. if you have other web services running on the same machine, it'd be better to use [[Reverse Proxy]] .

Visit mai.yourdomain.com. Login to admin with the password. Click Sign in Admin to enter admin dashboard.
<img width="364" height="230" alt="Pasted image 20251014015645" src="https://github.com/user-attachments/assets/b55ec19f-5e4c-40a7-bd39-7bd43e62573e" />


Click Mail Domains in the sidebar

<img width="245" height="495" alt="Pasted image 20251014015944" src="https://github.com/user-attachments/assets/25696d95-34be-46f5-b592-ef52d712d807" />


Click "details" under Actions column. Create new domain if it's not appeared in the list.
<img width="1917" height="947" alt="Pasted image 20251014020320" src="https://github.com/user-attachments/assets/e5a40b77-812b-42d1-965b-ec817e023e41" />


generate keys or any other domain records you wish to add. Then you can either copy past or download zone file to port to the DNS. This will take few minuintes.
<img width="2520" height="917" alt="Pasted image 20251014020610" src="https://github.com/user-attachments/assets/d1feb8ab-f80a-4911-b094-ba86e6c75ff4" />

## 3. Add user/ using third-party client.

before we add user, we can go to Webmail in the sidebar sending an email with admin account to test the reachability.
<img width="504" height="364" alt="Pasted image 20251014021738" src="https://github.com/user-attachments/assets/f309553d-3fa6-4cbe-8333-f5cb8b814f54" />


Under Domain List as previous step, Click Users under Manage column. then you will see New user bottom at right-top corner.
<img width="2025" height="662" alt="Pasted image 20251014022036" src="https://github.com/user-attachments/assets/f8a8f6e4-4f1f-4d65-a764-5d2ab9235ed8" />

You can use third-party client using these information listed in Client setup. 
<img width="1305" height="1231" alt="Pasted image 20251014022404" src="https://github.com/user-attachments/assets/47088f4b-a907-4be9-9498-ba21a7a613ba" />


## 4. test and improvement

I sent a test message on https://www.mail-tester.com/ 

Here's the result:
<img width="1463" height="1286" alt="Pasted image 20251014022933" src="https://github.com/user-attachments/assets/f951abf4-5b9f-4e3c-adc3-874fef8543ac" />

A .top TLD is cheap but very limited in some cases. Because many phishing website are using this cheap TLD.
<img width="621" height="211" alt="Pasted image 20251014023248" src="https://github.com/user-attachments/assets/7c8ea129-3c73-4531-a999-787982d9f01c" />

no rDNS. Let's setup one
<img width="1318" height="121" alt="Pasted image 20251014023335" src="https://github.com/user-attachments/assets/7674f05e-f16d-470b-b113-780a8eff3259" />

Typically, a reverse DNS (PTR) record must be configured with the provider that manages the IP address — in this case, my VPS provider.

They have a pretty straight forward interface. Just type in your mail. subdomain and ip address.
<img width="1697" height="776" alt="Pasted image 20251014125037" src="https://github.com/user-attachments/assets/931515c3-6b12-441b-8644-b38b1b16068d" />

Let's test again. We got a much higher point.
<img width="1118" height="382" alt="Pasted image 20251014125156" src="https://github.com/user-attachments/assets/bcc436a4-5d03-4e35-bbee-9d4563e77827" />

Test it with my outlook inbox. it was still identified as a junk. maybe because of the .top TLD.

<img width="844" height="258" alt="Pasted image 20251014125455" src="https://github.com/user-attachments/assets/6a15a559-dd49-4d84-925f-412ac44d4693" />

