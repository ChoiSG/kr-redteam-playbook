# Internet Censorship

Internet Censorship is now becoming more common around the world. Each country has their own reasons to seek to limit outside influences within there country.

### 

**WARNING: THIS POST IS ONLY FOR EDUCATIONAL PURPOSE.**[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#warning-this-post-is-only-for-educational-purpose.)

[Internet Censorship and surveillance by country (2017)](https://commons.wikimedia.org/wiki/File:Internet_Censorship_and_Surveillance_World_Map.svg#/media/File:Internet_Censorship_and_Surveillance_World_Map.svg)​

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Internet_Censorship_and_Surveillance_World_Map.svg/1200px-Internet_Censorship_and_Surveillance_World_Map.svg.png)

Internet Censorship and Surveillance World Map.svg

### 

[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#undefined)

One argument around internet censorship can be equated to the unstoppable force paradox which sates "What happens when an unstoppable force meets an immovable objections?". Every year, the internet censorship of each country continue to improve and getting stronger. One reason for the continuous improvement is due to censorship continuously being override by users who know how to bypass the censorship system and prevent themselves from being tracked by governments. So, who will win, the spear ( internet users, who always find out how to bypass the censorship) that can pierce any shield, or the shield ( Governments) can defend from all spear attacks?

The following pictures are the warning of censored content displayed by Australian, South Korean, French and Malaysian governments.

​

​

Before we start, let's talk about how clients request web pages from servers. In Client-Server network, the client establishes a connection to the server over a network. The client sends a request, and the server returns a response.

![](https://hexpertlab.files.wordpress.com/2018/08/client_server1.jpg)

Handshake between Client and Server

Clients and servers exchange messages in a request-response messaging pattern. A server provides service to one or many clients, and the clients initiate requests for such services.

**Note: The following scenarios are not about "circumvention" tools such as VPN, proxies like shadowsocks, Tor, sneakernet etc but are the basis of different technical censorship methods.**

### 

0x01 Internet Censorship by DNS filtering[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#0x01-internet-censorship-by-dns-filtering)

![](https://hexpertlab.files.wordpress.com/2018/08/http.jpg)

Figure 1. Connection between Client and Server

As you can see in Figure 1 above , this is a basis of how the client is communicating to the server. Let's say you want to connect to hexpertlab.com. Once your request to the browser is made, the browser will talk to the DNS server inside/outside your network (depending on which DNS server you are using for ) and the DNS server will response back with the IP address of hexpertlab.com stored in its database. Now your browser knows the IP address of hexpertlab.com, then 4) it will send another request to the hexpertlab website server, asking for the default page /root. Finally, the server will response back to the browser with the contents of default page, and boom! now you can display the website on your browser.

![](https://hexpertlab.files.wordpress.com/2018/08/waning1.jpg)

Figure 2. Performing Internet Censorship by DNS filtering

For any HTTP (unencrypted/plaintext traffic) is visible between your browser and the website server, it will block the blacklist domain www.hexpertlab.com by filtering out the domain name.

So any HTTP traffic is visible so it's not hard to intercept the traffic in between. How about HTTPS? You may realize there's no point of intercepting HTTPS traffic because it's already encrypted. However, can you still apply Internet Censorship against HTTPS? The answer is **YES**.

### 

0x02 Internet Censorship against HTTPS Connection[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#0x02-internet-censorship-against-https-connection)

As you can see in Figure 3, Enforcing HTTPS on your website will let you bypass the internet censorship easily. Any data between your browser and the website server (hexpert server 132.192.27.1) is encrypted via SSL certificate. Even though the censorship finds out and intercept the traffic, they do not know which endpoint you are visiting.

![](https://hexpertlab.files.wordpress.com/2018/08/https1.jpg)

Figure 3. Bypassing Internet Censorship by using HTTPS connection

So, does that mean it is impossible to filter out HTTPS connection? Governments can still apply internet censorship to HTTPS connection, by having internet service providers (ISPs) monitor the Server Name Indication (SNI).

![](https://hexpertlab.files.wordpress.com/2018/08/sni.jpg)

Figure 4. Performing Internet Censorship against HTTPS connection

​

SNI(Server Name Indication) is an extension of the Transport Layer Security (TLS) protocol, which can be monitor by ISPs if applicable. Although TLS encrypts the content of the traffic, neither it nor SNI encrypts the requested server name.

**Note**: Filtering through the SNI fields of internet traffic is quite different than deep packet inspection or blocking thousands and thousands of IP addresses. Let me cover this topic DPI in the article "Great firewall of China"

There has been some discussion going on about SNI encryption. https://datatracker.ietf.org/doc/draft-ietf-tls-sni-encryption/?include_text=1

### 

0x03 Bypassing Censorship using Third-party DNS servers[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#0x03-bypassing-censorship-using-third-party-dns-servers)

![](https://hexpertlab.files.wordpress.com/2018/08/blacklist.jpg)

Figure 5. Performing Internet Censorship by using ISP Partner DNS Server

​

As you can see in Figure 5, if your ISP coperates with the goverment(Political bias) and decide to monitor your HTTPS traffic, basically you have no luck.

However, you can choose not to use ISP DNS server but to have your own DNS server, as seen in Figure 6. By resolving domain name via any third-party DNS servers or perhaps having your own list of domains and IPs can possibly let you browse those filtered websites.

![](https://hexpertlab.files.wordpress.com/2018/08/changeisp-e1535541298688.jpg)

changeisp.jpg

### 

0x04 DNS over HTTPS[](https://dil15.gitbook.io/hexpertfightclub/~/changes/yiTNlxiCsMD9qGk0OEtN/miscell/internet-censorship/others#0x04-dns-over-https)

![](https://hexpertlab.files.wordpress.com/2018/08/dnsoverhttps.jpg)

dnsoverhttps

Figure 7. Bypassing Internet Censorship by using DNS over HTTPS

DNS over HTTPS is provided by Firefox and others, and also will be deployed in other browsers soon. I'll talk about this more in the next article "Great firewall of china"

​