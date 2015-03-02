# SSL Certificate configuration on Nginx
----------------

When configuring SSL certificates most of the steps are pretty straight forward. But there's a small step if you forget can lead to the following annoying error, even where your certificate is verified:

![image](https://s3.amazonaws.com/f.cl.ly/items/2R1t0u0O1s2z430P1X3d/Image%202015-03-02%20at%2014%3A44%3A05.png =300x)

Again, the steps to create the CSR are pretty straight forward. The following instructions asume that the certificate is one from **GoDaddy**, but I'm pretty sure that this applies to certificates from any other vendor.

### 1 - Generate the SSL request

You first need to create the key:

    # GoDaddy requires you to generate a 2048 key in size.
    openssl genrsa -out www.mysite.com.key 2048
    
Then your create the Certificate Signing Request or CSR:

    # The name of the files can be whatever you want, just be consistent.
    openssl req -new -key www.mysite.com.key -out www.mysite.com.csr

That will prompt you with a series of questions you need to answer:

    Country Name (2 letter code) [AU]:US
	State or Province Name (full name) [Some-State]:New York
	Locality Name (eg, city) []:Your town
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:Your Corporation
	Organizational Unit Name (eg, section) []:IT
	Common Name (eg, YOUR name) []:www.mysite.com
	Email Address []:someone@mysite.com

Just answer accordingly. The most important thing is the Common Name, it needs to be exactly the same you created the CSR for. So, if you created the CSR for `www.mysite.com`, enter that. If your are creating a CSR for a wildcard subdomain, then enter `*.mysite.com`.

After that, you should have to files: the key, `www.mysite.com.key` and the CSR, `www.mysite.com.csr`.

### 2 - Get your SSL Certificate.

Get your certificate. In this case, the certificate was bought on GoDaddy.com. I won't go on the steps to get the certificate, but I think for most vendors is almost the same. 

[GoDaddy SSL Certificates](https://uk.godaddy.com/ssl/ssl-certificates.aspx)

### 3 - Install the Certificate.

Once you get the certificate install it on your server. You should get a certificate and a bundle of authorities. Put those files somewhere in your server.

Let's say you uploaded the files to `/var/certificates/www.mysite.com.crt` and `/var/certificates/gd_bundle.crt`.

Now, in your web site configuration, or configurations, you need to point the `ssl_certificate` and `ssl_certificate_key` to the corresponding files, but **there's a trick here**.

Before you point the configuration settings to your brand new certificates you need to create an authorities chain in order for all the browsers to find your root certificate. Otherwise, you could find yourself facing the ugly error described above in some browsers.

### 4 - Combine them.

In the directory you uploaded the certificate and the bundle run the following:

    cat www.mysite.com.crt gd_bundle.crt > www.mysite.com_combined.crt
    
Now, the `www.mysite.com_combined.crt` is the one you have to point the Nginx configuration settings to.

    server {
        listen 443;
        server_name www.mysite.com;
        
        ssl on;
        ssl_certificate     /var/certificates/www.mysite.com_combined.crt;
        ssl_certificate_key /var/certificates/www.mysite.com.key;
        ...
    }

And then reload the configuration:

    sudo service nginx reload

And that's it. If the bundle is correct, in theory, all browsers can find your Certificate Authority (CA) and you should see that beautiful green `https://` in the URL bar of your browser.