# Windows Local Persistence

#### 🛡️ Persistence After Initial Foothold

* Once you've gained initial access to an internal network, your top priority should be to **maintain that access**.
* **Persistence** means setting up alternative methods to regain entry without needing to exploit the system again.
* It's one of the first and most crucial tasks after compromising a target.

***

#### 📱 Backdoored Device & Why Persistence Matters

* **Unstable exploits can be one-time use**: Some vulnerabilities may crash the service or app after exploitation, making repeated attempts impossible.
* **Hard-to-reproduce attack vectors**: For instance, if access was gained via a phishing campaign, recreating the exact conditions or success rate could be difficult or ineffective the second time.
* **Time pressure from defenders**: If your intrusion is noticed, patches may be applied quickly. You need a way in before the door closes.
* **Credentials are not reliable long-term**: Even if you steal a password hash, it could be rotated or changed, cutting off your access.
* **Stealthier methods are better**: Using creative persistence mechanisms can help evade detection and frustrate the defenders.
