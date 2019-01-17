# cli.openfaas.com
The installation script for the OpenFaaS CLI served by Netlify

Response Content-Type is application/x-shellscript. You should get a response header like this:

```python
import requests
r = requests.get('https://cli.openfaas.com')
r.headers
```

Header of the response:
```json
{
	"Accept-Ranges": "bytes",
	"Cache-Control": "public, max-age=0, must-revalidate",
	"Content-Type": "application/x-shellscript",
	"Date": "Sun, 13 Jan 2019 10:07:00 GMT",
	"Etag": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-ssl-df",
	"Strict-Transport-Security": "max-age=31536000",
	"X-Nf-Srv-Version": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
	"Content-Encoding": "gzip",
	"Content-Length": "1159",
	"Age": "124108",
	"Connection": "keep-alive",
	"Server": "Netlify",
	"Vary": "Accept-Encoding",
	"X-NF-Request-ID": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxxxx"
}
```

What happened if you try to receive the faas-cli script installion script via http? The connection will be upgraded to https and redirects you to https://cli.openfaas.com/get.sh

You can force to ignore redirection by following command:
```python
r = requests.get('http://cli.openfaas.com', allow_redirects=False)
```

You will get the redirected url via:
```python
r.next.url
```

The content you will received Is maintained in the following git repository: https://github.com/openfaas/cli.openfaas.com

Please keep in mind, all connections are redirected from "/" to "/get.sh" by a 301 http code.


##### faas-cli installation process is described [here](https://docs.openfaas.com/cli/install/).

The following wil describe how it works in detail.

```python
curl -sSL https://cli.openfaas.com | sh
```

First -s stands for silent and -S to show errors, -L ensures that the redirection by the 301 http code is accepted.
The pipe will pass the received content (get.sh script) to your shell and it will be executed.
[Here](https://curl.haxx.se/docs/manpage.html) you will find more about curl. 

##### The get.sh script in detail:

*1-part of the script checks faas-cli version:*
- Get latest version number from OpenFaaS repository.

*2-part checks if faas-cli is already installed (hasCli function)*
- Script will wait for 1s to chancel the installation of the latest cli version
- It will also check if curl is installed (ToDo: is this really need, because we received script via curl)

*3-part downloads cli-faas from github (getPackage)*
- Set architecture variable "arch", this is done by uname -a command
- Supported architectures are -armhf(armv6l and armv7l), -arm64(aarch64), -darwin(Mac? Darwin) or empty suffix for AMD64 and x86 anything else
- Target file path is set and old file will be removed if it exists 
- Download will be started via curl
- Hash will be checked (checkHash function, it will use sha256)
- Downloaded file will be marked as executable (chmod +x $targetFile)

##### Please check if you are running the installation as unprivileged user:
```bash
sudo cp faas-cli$suffix /usr/local/bin/faas-cli
sudo ln -sf /usr/local/bin/faas-cli /usr/local/bin/faas
```

faas-cli will always be copied to:
```bash
/usr/local/bin/faas-cli
```

A symbolic link will be set:
```bash
ln -s /usr/local/bin/faas-cli /usr/local/bin/faas
```

At the end faas-cli version shows the installed version.
