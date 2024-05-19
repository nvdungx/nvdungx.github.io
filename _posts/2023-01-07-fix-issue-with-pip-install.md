---
layout: post
title: Fix issue with pip install
date: 2023-1-07 00:00:00
categories: [programming]
tags: [development environment, beginner]
last_modified_at: 2023-02-15
---
  When you using python pip to install package with in a proxy environment, it might fail with below warning.
{% highlight powershell %}
PS C:\Users> pip install py-pdf-parser[dev]
Collecting py-pdf-parser[dev]
  WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:992)'))': /packages/c8/6c/dde3b91b0cc433946c37c20acc0a48e8ce762c0e1b4e1b702eb7d0c31e35/py_pdf_parser-0.10.2-py3-none-any.whl
  WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:992)'))': /packages/c8/6c/dde3b91b0cc433946c37c20acc0a48e8ce762c0e1b4e1b702eb7d0c31e35/py_pdf_parser-0.10.2-py3-none-any.whl
{% endhighlight %}

  To fix it, you can create pip config file and add a list of trusted-host to pip config file as below:
{% highlight powershell %}
PS C:\Users> mkdir ~/pip
PS C:\Users> ni -Type File ~/pip/pip.ini
PS C:\Users> explorer ~/pip/
// copy below string into C:\Users\<username>\pip\pip.ini file, 
// input below content into the newly created file, beware of  invalid characters error
"[global]
trusted-host = pypi.python.org
               pypi.org
               files.pythonhosted.org"
{% endhighlight %}
