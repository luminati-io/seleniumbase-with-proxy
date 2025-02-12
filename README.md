# Using Proxies With SeleniumBase

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to set up SeleniumBase with authenticated proxies to bypass restrictions and enhance your web scraping success.

- [Why We Need SeleniumBase](#why-we-need-seleniumbase)
- [Getting Started](#getting-started)
- [Invoking The Test](#invoking-the-test)
    - [Without a Proxy](#without-a-proxy)
    - [Proxy Configuration](#proxy-configuration)
    - [Running With a Proxy](#running-with-a-proxy)
    - [Controlling Your Location](#controlling-your-location)
    - [Rotating Proxies](#rotating-proxies)
- [Conclusion](#conclusion)

## Why We Need SeleniumBase

Selenium doesn’t support authenticated proxies. Now that SeleniumWire has been deprecated for over a year already and hasn't had any updates in over two years, the solution is to use [SeleniumBase](https://seleniumbase.com/). The latter project was originally designed as a wrapper for running Selenium instances to perform web automation & testing with Python. However, for us, it allows running Selenium using an authenticated proxy.

## Getting Started

Start with installing SeleniumBase:

```bash
pip install seleniumbase
```

Now write a test case that will control Selenium and run the webdriver instance. The code below makes a request to the [IPinfo API](https://ipinfo.io/json). Once the script receives the JSON response, it parses the response and prints its contents to the console.

```python
from seleniumbase import BaseCase
from selenium.webdriver.common.by import By
import json

class ProxyTest(BaseCase):
   def test_proxy(self):
        #go to the site
        self.driver.get("https://ipinfo.io/json")
      
        #load the json response
        location_info = json.loads(self.driver.find_element(By.TAG_NAME, "body").text)

        #iterate through the dict and print its contents
        for k,v in location_info.items():
            print(f"{k}: {v}")
```

## Invoking The Test

Let's run the test with `pytest` rather than with `python`.

### Without a Proxy

To run the test script without a proxy, use the command below:

```bash
pytest proxy_test.py -s
```

The response will be similar to this one:

```
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item                                                                                                          

proxy_test.py ip: 23.28.108.255
hostname: d28-23-255-108.dim.wideopenwest.com
city: Westland
region: Michigan
country: US
loc: 42.3242,-83.4002
org: AS12083 WideOpenWest Finance LLC
postal: 48185
timezone: America/Detroit
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 1.01s ====================================================
```

### Proxy Configuration

Use the `--proxy` flag followed by the proxy URL. Here is the template:

```
--proxy=your_proxy_url:your_proxy_port
```

#### Free Proxy

The following example uses a [free proxy](https://brightdata.com/solutions/free-proxies) from Bright Data. The IP address is `155.54.239.64`, and we’re talking to it on port `80`.

```
--proxy=155.54.239.64:80
```

#### Authenticated Proxy

Authenticated proxies are handled the same way. Simply include your username and password in the URL:

```
proxy=<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_URL>:<PROXY_PORT>
```

#### Authenticated Proxy Types

The best authenticated proxies options are:

- [**Residential proxies**](https://brightdata.com/proxy-types/residential-proxies): use real user IPs which makes them ideal for bypassing bot detection.
- [**Datacenter proxies**](https://brightdata.com/proxy-types/datacenter-proxies): faster and more cost-effective but easier to detect.
- [**ISP proxies**](https://brightdata.com/proxy-types/isp-proxies): combine the benefits of both, offering speed with high trust levels.

### Running With a Proxy

The example below is set up to run with one of our proxies at Bright Data. Be sure to replace the username, zone name, and password with your own credentials.

```bash
pytest proxy_test.py --proxy=brd-customer-<YOUR-USERNAME>-zone-<YOUR-ZONE-NAME>:<YOUR-PASSWORD>@brd.superproxy.io:33335 -s
```

Running it results in the following output:

```
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item

proxy_test.py ip: 144.202.4.246
hostname: 144-202-4-246.lum-int.io
city: Piscataway
region: New Jersey
country: US
loc: 40.4993,-74.3990
org: AS20473 The Constant Company, LLC
postal: 08854
timezone: America/New_York
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 3.25s ====================================================
```

### Controlling Your Location

With Bright Data's proxies, you can choose your location by using the `country` flag and a two-letter country code:

```bash
pytest proxy_test.py --proxy=brd-customer-<YOUR-USERNAME>-zone-<YOUR-ZONE-NAME>:<YOUR-PASSWORD>-country-es@brd.superproxy.io:33335 -s
```

When you use `es` (Spain) as your country code, you get routed through a proxy in Spain. You can verify this in the output below.

```
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item

proxy_test.py ip: 176.119.14.158
city: Paracuellos de Jarama
region: Madrid
country: ES
loc: 40.5035,-3.5278
org: AS203020 HostRoyale Technologies Pvt Ltd
postal: 28860
timezone: Europe/Madrid
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 3.98s ====================================================
```

Please refer to the [geolocation documentation](https://docs.brightdata.com/api-reference/proxy/geolocation-targeting) for details.

### Rotating Proxies

The code below uses a set of country codes, but these can easily be replaced with actual proxy IPs. `countries` holds the list of country codes. Next, we iterate through them and run our proxy test with all four country codes.

- `us`: United States
- `es`: Spain
- `il`: Israel
- `gb`: Great Britain

```python
import subprocess

#list of country codes
countries = [
    "us",
    "es",
    "il",
    "gb",
]

#iterate through the countries and make a shell command for each one
for country in countries:
    command = f"pytest proxy_test.py --proxy=brd-customer-<YOUR-USERNAME>-<YOUR-ZONE-NAME>-country-{country}:[email protected]:33335 -s"

    #run the shell command
    subprocess.run(command, shell=True)
```

You can run this as a regular Python file:

```bash
python rotate_proxies.py
```

The output should be similar to this:

```
(Linux uses --headless by default. To override, use --headed / --gui. For Xvfb mode instead, use --xvfb. Or you can hide this info by using --headless / --headless2 / --uc.)
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item                                                                                                          

proxy_test.py ip: 164.90.142.33
city: Clifton
region: New Jersey
country: US
loc: 40.8344,-74.1377
org: AS14061 DigitalOcean, LLC
postal: 07014
timezone: America/New_York
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 3.84s ====================================================
(Linux uses --headless by default. To override, use --headed / --gui. For Xvfb mode instead, use --xvfb. Or you can hide this info by using --headless / --headless2 / --uc.)
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item                                                                                                          

proxy_test.py ip: 5.180.9.15
city: Madrid
region: Madrid
country: ES
loc: 40.4066,-3.6724
org: AS203020 HostRoyale Technologies Pvt Ltd
postal: 28007
timezone: Europe/Madrid
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 3.60s ====================================================
(Linux uses --headless by default. To override, use --headed / --gui. For Xvfb mode instead, use --xvfb. Or you can hide this info by using --headless / --headless2 / --uc.)
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item                                                                                                          

proxy_test.py ip: 64.79.233.151
city: Tel Aviv
region: Tel Aviv
country: IL
loc: 32.0809,34.7806
org: AS9009 M247 Europe SRL
timezone: Asia/Jerusalem
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 3.36s ====================================================
(Linux uses --headless by default. To override, use --headed / --gui. For Xvfb mode instead, use --xvfb. Or you can hide this info by using --headless / --headless2 / --uc.)
=================================================== test session starts ===================================================
platform linux -- Python 3.10.12, pytest-8.3.4, pluggy-1.5.0
rootdir: /home/nultinator/clients/bright-data/seleniumbase-proxies
plugins: html-4.0.2, metadata-3.1.1, anyio-4.0.0, seleniumbase-4.34.6, ordering-0.6, rerunfailures-15.0, xdist-3.6.1
collected 1 item                                                                                                          

proxy_test.py ip: 185.37.3.107
city: London
region: England
country: GB
loc: 51.5085,-0.1257
org: AS9009 M247 Europe SRL
postal: E1W
timezone: Europe/London
readme: https://ipinfo.io/missingauth
.

==================================================== 1 passed in 2.90s ====================================================
```

## Conclusion

SeleniumBase opens up many web scraping capabilities in Selenium. Unlock the full potential of Selenium-based scraping with Bright Data’s [industry-leading proxy services](https://brightdata.com/proxy-types). Start your free trial today!
