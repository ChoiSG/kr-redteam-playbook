# Table of contents

* [레드팀 플레이북](README.md)
* [레드팀이란](what-even-is-redteam.md)
* [레드팀 글로벌 동향 (2024)](redteam-global-trend-2024.md)

## 🚧 인프라 (Infrastructure)

* [개념](infrastructure/concepts.md)
* [예시 인프라](infrastructure/example-infra.md)
* [팀 서버 - Sliver](infrastructure/sliver/README.md)
  * [스테이저 (Stager) 사용](infrastructure/sliver/stager.md)
* [도메인 분류와 신뢰도](infrastructure/domain-categorization-reputation.md)
* [HTTP 리다이렉터](infrastructure/http-redirector.md)
* [HTTPS 리다이렉터](infrastructure/https-redirector.md)
* [SMTP Gophish + Mail](infrastructure/smtp-do.md)
* [SMTP Gophish + ESP](infrastructure/smtp-aws-zoho.md)
* [SMTP Gophish + Relay + ESP](infrastructure/smtp-toolkit-relay-esp.md)
* [인프라 구축 자동화](infrastructure/infra-automation/README.md)
  * [테라폼 (Terraform)](infrastructure/infra-automation/terraform.md)
  * [SMTP 테라폼 자동화](infrastructure/infra-automation/mail-terraform.md)
  * [HTTPS 리다이렉터 자동화 (AWS)](infrastructure/infra-automation/https-redir-automation.md)
* [old-네뷸라를 이용한 인프라 구축](infrastructure/old/README.md)
  * [도메인과 리다이렉터 설정](infrastructure/old/domain-redirector-config.md)
  * [중립 공간 (클라우드) 설정](infrastructure/old/neutral-area-cloud-config.md)
  * [네뷸라 (Nebula)](infrastructure/old/nebula.md)
  * [네뷸라 설정](infrastructure/old/nebula-config.md)
* [도메인 프론팅 (Domain Fronting)](infrastructure/domain-fronting.md)
* [도메인 프론팅 - Azure CDN](infrastructure/azure-cdn.md)

## 🔎 초기 정찰 <a href="#initial-recon" id="initial-recon"></a>

* [개념](initial-recon/concept.md)
* [타겟 발견](initial-recon/target-discovery.md)
* [OSINT](initial-recon/osint/README.md)
  * [작전보안](initial-recon/osint/opsec.md)
  * [자산 정보 수집](initial-recon/osint/asset-discovery.md)
  * [구글 도킹](initial-recon/osint/google-dorking.md)

## ⚔️ 초기 침투 (Initial Access)

* [개념](initial-access/concepts.md)
* [피싱 첨부파일](initial-access/phish-attachments/README.md)
  * [오피스 VBA 매크로](initial-access/phish-attachments/vba-macros.md)
  * [XLM Excel 4.0 매크로](initial-access/phish-attachments/xlm-excel-40.md)
  * [원격 템플렛 인젝션](initial-access/phish-attachments/remote-template-injection.md)
  * [VBA Stomping](initial-access/phish-attachments/vba-stomping.md)
  * [HTA](initial-access/phish-attachments/hta.md)
  * [LNK](initial-access/phish-attachments/lnk.md)
  * [ISO](initial-access/phish-attachments/iso.md)
  * [VBA Purging - TODO](initial-access/phish-attachments/vba-purging.md)
  * [DotNetToJS - TODO](initial-access/phish-attachments/dotnettojs.md)
  * [Follina - TODO](initial-access/phish-attachments/follina.md)
* [HTML 스머글링 (Smuggling)](initial-access/html-smuggling.md)
* [피싱 - AitM (Adversary in the Middle)](initial-access/aitm.md)
* [Living Off Trusted Sites (LOTS)](initial-access/living-off-trusted-sites.md)

## 🐳 정보 수집 - 내부망 <a href="#enumeration" id="enumeration"></a>

* [개념](enumeration/concepts.md)
* [로컬 호스트 정보 수집](enumeration/local-host-enumeration.md)
* [블러드하운드](enumeration/bloodhound.md)
* [SMB 쉐어 수집](enumeration/smb-share.md)
* [정보 수집 - 파워쉘](enumeration/powershell.md)
* [정보 수집 - C# - TODO](enumeration/csharp.md)
* [커버로스 유저 이름 정보수집](enumeration/kerberos-username-enumeration.md)
* [CME - 호스트이름과 IP주소](enumeration/cme-hostname-ip.md)
* [LDAP Anonymous Bind](enumeration/ldap-anonymous-bind.md)

## 🐴 실행 (Execution)

* [개념](execution/concepts.md)
* [파워쉘](execution/powershell/README.md)
  * [인메모리 실행](execution/powershell/in-memory-execution.md)
  * [C# 실행](execution/powershell/csharp-execution.md)
  * [윈도우 API 실행](execution/powershell/winapi-execution.md)
* [LOLBAS](execution/lolbas.md)
* [Native API - TODO](execution/native-api.md)

## 🙃 지속성 (Persistence)

* [개념](persistence/concepts.md)
* [골든 티켓 (Golden Ticket)](persistence/golden-ticket.md)
* [DLL 사이드로딩 (DLL Side-Loading)](persistence/dll-sideloading.md)
* [DLL Search Order Hijacking - TODO](persistence/dll-hijacking.md)
* [레지스트리 / 스타트업 폴더](persistence/registry-startup-folder.md)

## ⬆️ 권한 상승 <a href="#privilege-escalation" id="privilege-escalation"></a>

* [개념](privilege-escalation/concepts.md)
* [AD 권한 상승](privilege-escalation/ad/README.md)
  * [Active Directory Certificate Services (ADCS)](privilege-escalation/ad/adcs/README.md)
    * [ESC1](privilege-escalation/ad/adcs/esc1.md)
    * [ESC8](privilege-escalation/ad/adcs/esc8.md)
  * [Shadow Credentials](privilege-escalation/ad/shadow-credentials.md)
  * [noPac](privilege-escalation/ad/nopac.md)
  * [Kerberoasting](privilege-escalation/ad/kerberoasting.md)
  * [AS-REP Roasting](privilege-escalation/ad/asrep-roasting.md)
  * [DHCPv6 포이즈닝](privilege-escalation/ad/dhcpv6.md)
  * [Resource-Based Constrained Delegation (RBCD)](privilege-escalation/ad/rbcd.md)
  * [SCCM](privilege-escalation/ad/sccm.md)
* [AD-DACL](privilege-escalation/ad-dacl/README.md)
  * [AddAllowedToAct](privilege-escalation/ad-dacl/addallowedtoact.md)
  * [AddKeyCredentialLink](privilege-escalation/ad-dacl/addkeycredentiallink.md)
  * [GenericAll](privilege-escalation/ad-dacl/genericall.md)
  * [GenericWrite](privilege-escalation/ad-dacl/genericwrite.md)
  * [WriteDACL](privilege-escalation/ad-dacl/writedacl.md)
  * [AllExtendedRights](privilege-escalation/ad-dacl/allextendedrights.md)
  * [WriteAccountRestrictions](privilege-escalation/ad-dacl/writeaccountrestrictions.md)
  * [WriteOwner](privilege-escalation/ad-dacl/writeowner.md)
  * [AddMember](privilege-escalation/ad-dacl/addmember.md)
* [로컬 권한 상승 - TODO](privilege-escalation/local/README.md)
  * [잘못된 서비스 설정](privilege-escalation/local/misconfigured-services.md)
  * [Unquoted Service Path](privilege-escalation/local/unquoted-service-path.md)
  * [Always Install Elevated](privilege-escalation/local/always-install-elevated.md)
  * [PrintNightmare](privilege-escalation/local/printnightmare.md)

## 🐍 보안 우회 (Defense Evasion)

* [쉘코드 암호화](defense-evasion/shellcode-encryption.md)
* [런타임 다이나믹 링킹 (Run-time Dynamic Linking)](defense-evasion/run-time-dynamic-linking.md)
* [AMSI 우회](defense-evasion/amsi-bypass.md)
* [유저랜드 후킹 - 역사](defense-evasion/userland-hooking-history.md)
* [유저랜드 커널랜드 윈도우API 개념](defense-evasion/ring0-ring3-winapi.md)
* [유저랜드 후킹](defense-evasion/userland-hooking.md)
* [DInvoke - 시스템 콜](defense-evasion/dinvoke-syscall.md)
* [페이로드 크기](defense-evasion/payload-size.md)
* [가변적 C2 프로필](defense-evasion/malleable-c2-profile.md)
* [프로세스 인젝션](defense-evasion/process-injection/README.md)
  * [CreateRemoteThread](defense-evasion/process-injection/createremotethread.md)
  * [NtMapViewOfSection](defense-evasion/process-injection/ntmapviewofsection.md)
* [간단 디펜더 우회 - 쉘코드](defense-evasion/simple-defender-bypass.md)
* [간단 디펜더 우회 - C#](defense-evasion/simple-defender-bypass-csharp.md)
* [MSIExec](defense-evasion/msiexec.md)
* [다형성 코드](defense-evasion/polymorphic-code.md)

## 👿 후속 공격 (Post Exploitation)

* [후속 공격의 진화](post-exploitation/evolution-of-postex.md)
* [External SID Golden Ticket](post-exploitation/external-sid-golden-ticket.md)

## 🎭 계정 정보 탈취 (Credential Access)

* [커버로스](credential-access/kerberos/README.md)
  * [커버로스팅 (Kerberoasting)](credential-access/kerberos/kerberoasting.md)
  * [AS-Rep Roasting](credential-access/kerberos/as-rep-roasting.md)
* [비밀번호 스프레이 공격](credential-access/password-spraying.md)
* [LLMNR/NBT-NS 포이즈닝](credential-access/llmnr-nbtns-poisoning.md)
* [NTLM 릴레이 (NTLM Relay)](credential-access/ntlm-relay/README.md)
  * [SMB to SMB](credential-access/ntlm-relay/smb-to-smb.md)
  * [SMB to LDAP/S](credential-access/ntlm-relay/smb-to-ldap-s.md)
  * [HTTP to LDAP](credential-access/ntlm-relay/http-to-ldap.md)
  * [SMB to HTTP](credential-access/ntlm-relay/smb-to-http.md)
  * [SMB to SCCM](credential-access/ntlm-relay/smb-to-sccm.md)
* [강제 인증 (Authentication Coercion)](credential-access/authentication-coercion/README.md)
  * [MS-RPRN - Printerbug / Print Spooler](credential-access/authentication-coercion/ms-rprn.md)
  * [MS-EFSRPC - Petitpotam](credential-access/authentication-coercion/ms-efsrpc.md)
  * [MS-FSRVP - ShadowCoerce](credential-access/authentication-coercion/ms-fsrvp.md)
  * [MS-DFSNM - DFSCoerce](credential-access/authentication-coercion/ms-dfsnm.md)
* [NTLM 다운그레이드](credential-access/ntlm-downgrade.md)
* [DHCPv6 포이즈닝](credential-access/dhcpv6.md)
* [LAPS - TODO](credential-access/laps-todo.md)
* [DCSync](credential-access/dcsync.md)
* [DPAPI](credential-access/dpapi.md)

## ↔️ 횡적 이동 (Lateral Movement)

* [개념](lateral-movement/concepts.md)
* [Pass-the-Hash](lateral-movement/pass-the-hash.md)
* [SMB 와 PsExec](lateral-movement/smb-psexec.md)
* [WMI](lateral-movement/wmi.md)
* [WinRM / Powershell Remoting](lateral-movement/winrm-ps-remoting.md)
* [RDP](lateral-movement/rdp.md)
* [SSH Port Forwarding & Tunneling](lateral-movement/ssh-tunnel.md)
* [Dynamic Port Forwarding & ProxyChains](lateral-movement/dynamic-port-fowarding.md)
* [네트워크 피버팅 - 툴](lateral-movement/pivoting-tools.md)
* [Network Pivoting (피벗) - TODO](lateral-movement/pivoting.md)

## 🏭 최신 취약점을 응용한 웹&모바일 해킹 과정 따라하기\&CTF 문제풀이 <a href="#critical-info-infrastructure" id="critical-info-infrastructure"></a>

* [01-계정관리](critical-info-infrastructure/01-account-management/README.md)
  * [통신 과정 흐름 훑어보기](critical-info-infrastructure/01-account-management/undefined.md)
  * [첫번쩨, payload설정 방법](critical-info-infrastructure/01-account-management/payload.md)
  * [두번째, 세션을 유지하는 방법](critical-info-infrastructure/01-account-management/undefined-1.md)
  * [CSRF Token에 관하여](critical-info-infrastructure/01-account-management/csrf-token.md)
  * [세번째, OWASP Juice Shop 로그인 페이지에서의 통신 흐름과 구조 알아보기](critical-info-infrastructure/01-account-management/owasp-juice-shop.md)

## 개념 <a href="#general-concepts" id="general-concepts"></a>

* [윈도우 사용자 인증](general-concepts/windows-authentication/README.md)
  * [NTLM 인증](general-concepts/windows-authentication/ntlm.md)
  * [커버로스 (Kerberos) 인증 - TODO](general-concepts/windows-authentication/kerberos-authentication.md)
  * [ADCS 인증서 기반 인증](general-concepts/windows-authentication/adcs-auth.md)
* [AD 관련 용어 해설](general-concepts/glossary.md)

## 실 공격 TTP와 대응방안 - TODO <a href="#real-attack-ttp-and-mitigations" id="real-attack-ttp-and-mitigations"></a>

* [개념](real-attack-ttp-and-mitigations/concepts.md)

## 🧑‍🔬 홈 랩 (Home lab) <a href="#homelab" id="homelab"></a>

* [시스몬 (sysmon) 설치](homelab/installing-sysmon.md)
* [SIEM과 EDR 솔루션 설치](homelab/edr.md)
* [취약한 랩을 위한 설정 커맨드](homelab/homelab-misconfigurations.md)
* [말웨어 자동 분석툴 Cuckoo 샌드박스 설치](homelab/cuckoo.md)

## 기본 개념 <a href="#basic-concepts" id="basic-concepts"></a>

* [DNS](basic-concepts/dns/README.md)
  * [사이버 보안에 필요한 DNS 개념 편 #1](basic-concepts/dns/dns-concepts.md)
  * [해킹에 필요한 DNS 공격 & 우회 기법 편 #2](basic-concepts/dns/dns-and-2.md)
  * [방어에 필요한 DNS 탐지 편 #3](basic-concepts/dns/dns-3.md)

## 🎅 MISC

* [Host-based 공격 CheatSheet (FEAT. OSCP)](misc/host-based-cheatsheet-feat.-oscp/README.md)
  * [Enumeration (정보 수집 및 열거)](misc/host-based-cheatsheet-feat.-oscp/enumeration.md)
  * [Exploitation (공격)](misc/host-based-cheatsheet-feat.-oscp/exploitation.md)
  * [Post Exploitation (후속공격)](misc/host-based-cheatsheet-feat.-oscp/post-exploitation.md)
* [Changelog](misc/changelog.md)
* [기여하는 방법](misc/contributions.md)
* [레퍼런스와 크레딧](misc/undefined.md)
* [C# snippets](misc/csharp-snippets.md)
* [winapi 리스트](misc/winapi-list/README.md)
  * [original notes from obsidian](misc/winapi-list/original-notes-from-obsidian.md)
* [파워쉘 원라이너 (oneliner)](misc/powershell-oneliners.md)

## 📖 Personal-Research

* [README-and-Template](personal-research/readme-and-template/README.md)
  * [HellsGate](personal-research/readme-and-template/hellsgate.md)
  * [HalosGate](personal-research/readme-and-template/halosgate.md)
  * [FreshyCall](personal-research/readme-and-template/freshycall.md)
