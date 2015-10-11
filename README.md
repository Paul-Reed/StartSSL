##_Shamelessly copied from [Eric Mill's](https://konklone.com/post/switch-to-https-now-for-free) excellent guide!_
_Eric's guide was helpful beyond words, and as I have to renew the SSL certificate annually, I needed to ensure that I can continue to access it..._

A quick overview: to use HTTPS on the web today, you need to obtain a certificate file that's signed by a company that browsers trust. Once you have it, you tell your web server where it is, where your associated private key is, and open up port 443 for business. You don't necessarily have to be a professional software developer to do this, but you do need to be **okay with the command line**, and comfortable configuring **a web server you control**.

Most certificates cost money, but at Micah Lee's [suggestion](https://twitter.com/micahflee/status/368163493049933824), I used [StartSSL](https://www.startssl.com). They're who the [EFF](https://www.eff.org/) uses, and **their basic certificates for individuals are free**.

There are two things that could cost you money. One is that if your site is commercial in nature, they'll ask you to pay for a higher level certificate.

More importantly, if your certificate needs to be revoked someday, StartCom will [charge you a $30 fee](https://www.startssl.com/?app=25#72). While revocation has generally been rare, the [Heartbleed](http://heartbleed.com/) exploit is an example where a huge portion of the Internet had to revoke their keys. For some people who had issued a large number of free certificates, this turned out to be expensive.

Still, StartCom makes getting started with SSL simple and inexpensive. Their website is difficult to use at first — especially if you're new to the concepts and terminology behind SSL certificates (like I was). Fortunately, it's not actually that hard; it's just a lot of small steps.

Below, we'll go step by step through signing up with StartSSL and creating your certificate. We'll also cover installing it via nginx, but you can use the certificate with whatever web server you want.

## Register with StartSSL

To get started, [visit their signup page](https://www.startssl.com/?app=11&action=regform) and enter your information.

<img src="/images/ssl-1-signup.png" class="block upper border" />

They'll email you a verification code. They tell you to **not close the tab** or navigate away from it, so just keep it open until you get the code, and can paste it in.

<img src="/images/ssl-2-signup-verify.png" class="block upper border" />

You'll need to wait for certification, but it should only take a few minutes. Once you're approved, they'll email you a special link and a verification code to type in.

That'll bring you to a screen to generate a private key. They're generating you this private key inside your browser, using the ["keygen" tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen). However, **this isn't the key you use to make your SSL certificate**. They're using it to create a separate "authentication certificate" that you will use to log in to StartSSL's control panel going forward. You'll make a separate certificate for your website later.

<img src="/images/ssl-3-auth-key-generate.png" class="block upper border" />

Finally, they'll ask you to "Install" the certificate:

<img src="/images/ssl-4-auth-cert.png" class="block upper border" />

Which installs your authentication certificate directly into your browser.

<img src="/images/ssl-5-auth-cert-installed.png" class="block upper border" />

If you're in Chrome, you should see this at the top of your browser window:

<img src="/images/ssl-6-auth-cert-chrome-install.png" class="block upper border" />

Again, this is just the certificate that identifies you by your email address and lets you log in to StartSSL going forward.

Now, we need to persuade StartSSL that we own the domain name we want to generate a new certificate for. From the control panel, click the "Validations Wizard" tab, and select "Domain Name Validation" from the dropdown.

<img src="/images/ssl-7-begin-domain.png" class="block upper border" />

Enter your domain name.

<img src="/images/ssl-8-choose-domain.png" class="block upper border" />

Next, you'll select an email address that StartSSL will use to verify you own the domain name.

As you can see, StartSSL will believe you own the domain if you control webmaster@, postmaster@, or hostmaster@ with the domain name, **OR** if you own the email address listed as part of the domain's registrant information (in my case, that's currently `konklone@gmail.com`). Choose an email address where you can receive mail.

<img src="/images/ssl-9-choose-domain-email.png" class="block upper border" />

They'll email you a validation code, which you can enter into the field to validate the domain.

<img src="/images/ssl-10-domain-validate.png" class="block upper border" />

<img src="/images/ssl-11-domain-done.png" class="block upper border" />

## Generating the certificate

Now that StartSSL knows who you are, and knows you own a domain, you can generate your certificate using a private key.

While StartSSL _can_ generate a private key for you — and their FAQ assures you they use only the [highest quality random numbers](https://www.startssl.com/?app=25#43) and [don't hold onto the key](https://www.startssl.com/?app=25#44) afterwards — it's better to create your own, as StartSSL never sees your private key.

To create a new 2048-bit RSA key, open up your terminal and run:

```bash
openssl genrsa -aes256 -out my-private-encrypted.key 2048
```

You'll be asked to choose a pass phrase. [Pick a good one](http://xkcd.com/936/), and **remember it**. This will generate an *encrypted* private key. If you ever need to transfer your key, via the network or anything else, use the encrypted version.

The next step is to decrypt it so that you can generate a "certificate signing request" with it. To decrypt your private key:

```bash
openssl rsa -in my-private-encrypted.key -out my-private-decrypted.key
```

Now, generate a certificate signing request. Don't worry about the details - all StartSSL cares about is the public key associated with the CSR.

```bash
openssl req -new -sha256 -key my-private-decrypted.key -out mydomain.com.csr
```

Go back to StartSSL's control panel and click the "Certificates Wizard" tab, and select "Web Server SSL/TLS Certificate" from the dropdown.

<img src="/images/ssl-12-cert-begin.png" class="block upper border" />

Since we generated our own private key, you can hit "**Skip**" here.

<img src="/images/ssl-13-cert-key-skip.png" class="block upper border" />

Then, paste in the contents of the .csr file we generated earlier.

<img src="/images/ssl-14-csr-enter.png" class="block upper border" />

If all goes well, it should say it received your certificate signing request.

<img src="/images/ssl-15-csr-received.png" class="block upper border" />

Now, choose the domain you validated earlier which you plan to use with the certificate.

<img src="/images/ssl-16-choose-domain.png" class="block upper border" />

It requires you to add a subdomain. I added "www" for mine.

<img src="/images/ssl-17-choose-subdomain.png" class="block upper border" />

It will ask you to confirm. If it looks right, hit "Continue".

<img src="/images/ssl-18-cert-ready.png" class="block upper border" />

<div class="callout">
<strong>Note</strong>: It's possible you'll get hit with a "Additional Check Required!" step here, where you wait for approval by email. It didn't happen to me the first time, but did the second time, and my approval arrived in ~30 minutes. Upon approval, you'll need to visit the "Tool Box" tab and visit "Retrieve Certificate" to get your cert.
</div>

That should do it — your certificate will appear in a text field for you to copy and paste into a file. Call it whatever you want, but the rest of the guide will refer to it as `mydomain.com.crt`.

## Creating the full certificate chain

(If you used <a href="https://sslmate.com">SSLMate</a>, you can <a href="#installing-the-certificates">skip this step</a>.)

Next, we're going to create the "certificate chain" that your web server will use. It contains your certificate, and StartSSL's intermediary certificate. (Including StartSSL's root cert is not necessary, because browsers ship with it already.) Download the intermediate certificate from StartSSL:

```bash
wget https://www.startssl.com/certs/class1/sha2/pem/sub.class1.server.sha2.ca.pem
```

Then concatenate your certificate with theirs:

```bash
cat mydomain.com.crt sub.class1.server.sha2.ca.pem > unified.crt
```

## Installing the certificates

If you have direct access to your web server and its nginx configuration, here's how to install your certificate. If you don't, check out [setup options for other common hosts](#setup-with-other-common-hosts) or [for Apache](https://www.insecure.ws/2013/10/11/ssltls-configuration-for-apache-mod_ssl/).

First, make sure **port 443 is open** on your web server. Many web hosts automatically keep this port open for you. If you're using Amazon Web Services, you'll need to make sure your instance's security group has port 443 open.

<div class="callout">
Also, take a look at David Zvenyach's <a href="https://github.com/vzvenyach/nginx-ssl">nginx-ssl</a>, a simple script to bootstrap the nginx/HTTPS process.
</div>

Finally, tell your web server about your unified certificate, and your decrypted private key. I use nginx — below is the bare minimum nginx configuration you need. It redirects all HTTP requests to HTTPS requests using a 301 permanent redirect, and points the server to the certificate and key.

<script src="https://gist.github.com/konklone/6416795.js"></script>

<div class="callout">
You can also check out <a href="https://gist.github.com/konklone/6532544">a more complete HTTPS nginx configuration</a> that turns on <a href="http://www.chromium.org/spdy/spdy-whitepaper">SPDY</a>, <a href="https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security">HSTS</a>, SSL <a href="https://code.google.com/p/sslyze/wiki/SessionResumption">session resumption</a>, <a href="http://en.wikipedia.org/wiki/OCSP_stapling">OCSP stapling</a>, and enables <a href="https://www.eff.org/deeplinks/2013/08/pushing-perfect-forward-secrecy-important-web-privacy-protection">Forward Secrecy</a>.
<br/><br/>
Qualys' SSL Labs offers an excellent <a href="https://www.ssllabs.com/ssltest/analyze.html">SSL testing tool</a> you can use to see how you're doing.
</div>

Now, ensure your nginx configuration is okay (this also verifies that the key and certificate are in working order):

```bash
sudo nginx -t
```

Then restart nginx:

```bash
sudo service nginx restart
```

Cross your fingers and try it out in your browser! If all goes well, the <img src="/images/ssl-0-lock.png" /> will be yours.

<div class="callout">
<strong>Important:</strong> the StartSSL free Class 1 certificate is good for just <strong>1 year</strong>. Don't forget to renew it before then! Set a calendar reminder or something!
</div>

## Setup with other common hosts

Many common hosts don't give you direct access to install a certificate yourself. In that case, you'll probably have to pay $$ for HTTPS, if it's possible at all.

If you use:

* **Heroku**, you'll need to pay $20/month for their [SSL add-on](https://addons.heroku.com/ssl), and then use it to [set up an SSL endpoint](https://devcenter.heroku.com/articles/ssl-endpoint). Check out Moncef Belyamani's [SSLMate + Heroku tutorial](http://www.moncefbelyamani.com/how-to-add-ssl-to-your-heroku-custom-domain-with-sslmate/) for some straightforward assistance.
* **Amazon S3**, as of March 2014 they support **[free SSL for custom domains](http://aws.amazon.com/cloudfront/custom-ssl-domains/)** via CloudFront. Bear in mind this requires [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication), which won't work for users running Internet Explorer on Windows XP or Android 2.x's default browser. It's also unsupported by Python 2.x. If that's a dealbreaker, then you'll have to pay an insane $600/month for a dedicated IP.
* **Apache**, check out [kang's blog post](https://www.insecure.ws/linux/apache_ssl.html) on making an Apache config that gets the A rating from Qualys.
* **Bytemark** and other servers using **[Symbiosis](https://www.bytemark.co.uk/hosting/symbiosis/)** for Debian support [simple SSL hosting as standard](http://symbiosis.bytemark.co.uk/docs/ch-ssl-hosting.html). Use the key generation guide above and name it `ssl.key`, following the Symbiosis documentation. Likewise, the certificate when generated should be `ssl.crt`. You'll also need StartSSL intermediate certificate that's mentioned above: [`sub.class1.server.sha2.ca.pem`](https://www.startssl.com/certs/class1/sha2/pem/sub.class1.server.sha2.ca.pem). Rename this to `ssl.bundle`.
* **Github Pages**, they offer [undocumented HTTPS support](https://konklone.com/post/github-pages-now-supports-https-so-use-it) for `*.github.io` domains. However, they offer no HTTPS support at all for custom domains, so for that you'll have to look elsewhere (see below).
* **Webfaction**, they [provide HTTPS support](http://docs.webfaction.com/user-guide/websites.html#secure-sites-https) at no extra charge. Go [Webfaction](https://www.webfaction.com/)!


If you need to **look elsewhere** because your host makes it too expensive or impossible to set up HTTPS, another option is to sign up for [CloudFlare](https://www.cloudflare.com/). You don't need to leave your current host to use them — they sit "in front" of your website and can speed it up in various ways. 

CloudFlare [offers HTTPS to anyone for free](https://blog.cloudflare.com/introducing-universal-ssl/), but there are two big catches:

* The free plan doesn't support clients using Windows XP or Python 2. To support older clients, you need a paid plan (which start at $20/month).
* **All** CloudFlare plans can only encrypt between the visitor and CloudFlare. To ensure that the connection is encrypted all the way from the visitor to your website, you'll need to install your own certificate on your own web server anyway and tell CloudFlare to use and validate that certificate.

The tradeoffs are yours to choose, and yours alone!

## Mixed Content Warnings

If your site is running on HTTPS, it's important to make sure all linked resources — images, stylesheets, JavaScript, etc. — are HTTPS too. If they're not, users' browsers will complain. Newer versions of Firefox will [outright block insecure content](https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/) on a secure page.

Fortunately, pretty much every major service with an embed code has an HTTPS version, and most (including Google Analytics and Typekit) handle it automatically. For others, you'll need to figure it out on a case by case basis.

Where you need to support both HTTP and HTTPS, use [protocol-relative URLs](http://billpatrianakos.me/blog/2013/04/18/protocol-relative-urls/) (starting URLs with `//domain.com`). They're supported just about everywhere except (of course) for IE6.

## Back up your keys and certificates

Don't forget to **back up your SSL certificate, and its encrypted private key**. I put them in a private git repository, and included a brief text file describing every other file, and the process or command that created it. Make sure to **record when your certificate expires**, and **set a calendar alarm** for that date!

You should also **back up your authentication certificate** that you use to log in to StartSSL. StartSSL's FAQ [has instructions](https://www.startssl.com/?app=25#4) — it's a .p12 file containing a cert + key that you export from your browser.

<div class="footer-text">
<strong>More Resources</strong>
<br/>
<a href="http://arstechnica.com/security/2009/12/how-to-get-set-with-a-secure-sertificate-for-free/">Glenn Fleishman's 2009 article</a> covers setting up HTTPS with Mac OS X and Apache.
<br/>
<a href="http://www.westphahl.net/blog/2012/01/03/setting-up-https-with-nginx-and-startssl/">Simon Westphahl's 2012 blog post</a> describes setting up HTTPS using the command line and nginx.
</div>
