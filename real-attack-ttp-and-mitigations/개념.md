# 개념

### 공격자 정보&#x20;

공격자 도메인: koreambtihealth.com&#x20;



### 타겟 정보&#x20;

* 타겟 내부망 도메인: choi.local&#x20;
* 타겟 피해자 유저: h.park@choi.local&#x20;



### TTP 정보&#x20;

1. 초기 침투&#x20;
   1. 피싱 이메일 + HTML 스머글링&#x20;
   2. 피해자 유저가 사내 내부망에서 자신의 개인 이메일 확인 -> 스머글링에 걸림&#x20;
   3. VBS Purged maldoc OR remote template injection maldoc OR DotNetToJscript (.js) file. If using maldocs, potentially use .zip protected? Or go for XSL + DotNetToJscript for executing xsl + Jscript&#x20;
      1. Embedded C# stager should use direct syscall (modified dinvoke) + AMSI/ETW bypass + ppid spoofing + process injection to inject shellcode into a remote process&#x20;
   4. Nope, using iso payload + lnk + dll sideloading (and hidden attributes) + sliver or Covenant.&#x20;
   5. Local enumeration + Domain enumeration&#x20;
   6. Kerberoasting service acocunt + Cracking&#x20;
   7. Gaining the service account with kerberoasting + accessing another workstation -> dumping LSASS using handlekatz from powershell + AMSI bypass&#x20;
   8. DA creds and yay!&#x20;



