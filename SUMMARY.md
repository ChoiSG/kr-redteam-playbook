# Table of contents

* [레드팀 플레이북](README.md)
* [레드팀이란](what-even-is-redteam.md)

## 인프라 (Infrastructure)

* [개념](infrastructure/concepts.md)
* [예시 인프라](infrastructure/example-infra.md)
* [팀 서버 - Sliver](infrastructure/sliver/README.md)
  * [스테이저 (Stager) 사용](infrastructure/sliver/stager.md)
* [도메인 분류와 신뢰도](infrastructure/domain-categorization-reputation.md)
* [중립 공간 (클라우드) 설정](infrastructure/neutral-area-cloud-config.md)
* [네뷸라 (Nebula)](infrastructure/nebula.md)
* [네뷸라 설정](infrastructure/nebula-config.md)
* [도메인과 리다이렉터 설정](infrastructure/domain-redirector-config.md)

## 공개 출처 정보 수집 (OSINT)

* [개념](osint/concepts.md)
* [작전보안](osint/opsec.md)
* [자산 정보 수집](osint/enumeration.md)
* [구글 도킹](osint/google-dorking.md)

## 초기 침투 (Initial Access)

* [개념](initial-access/concepts.md)
* [피싱 첨부파일](initial-access/phish-attachments/README.md)
  * [오피스 VBA 매크로](initial-access/phish-attachments/vba-macros.md)
  * [원격 템플렛 인젝션](initial-access/phish-attachments/remote-template-injection.md)
  * [VBA Stomping](initial-access/phish-attachments/vba-stomping.md)
  * [VBA Purging - TODO](initial-access/phish-attachments/vba-purging.md)
  * [LNK](initial-access/phish-attachments/lnk.md)
  * [ISO](initial-access/phish-attachments/iso.md)
  * [DotNetToJS](initial-access/phish-attachments/dotnettojs.md)
  * [HTA](initial-access/phish-attachments/hta.md)
  * [Follina](initial-access/phish-attachments/follina.md)
  * [XLM Excel 4.0 매크로](initial-access/phish-attachments/XLM-Excel-40.md)
* [HTML 스머글링 (Smuggling)](initial-access/html-smuggling.md)
* [피싱 - AitM (Adversary in the Middle)](initial-access/aitm.md)
* [Living Off Trusted Sites (LOTS)](initial-access/living-off-trusted-sites.md)

## 정보 수집 - 내부망 <a href="#enumeration" id="enumeration"></a>

* [개념](enumeration/concepts.md)
* [로컬 호스트 정보 수집](enumeration/local-host-enumeration.md)
* [블러드하운드](enumeration/bloodhound.md)
* [SMB 쉐어 수집](enumeration/smb-share.md)
* [정보 수집 - 파워쉘](enumeration/powershell.md)
* [정보 수집 - C# - TODO](enumeration/csharp.md)
* [커버로스 유저 이름 정보수집](enumeration/kerberos-username-enumeration.md)

## 실행 (Execution)

* [개념](execution/concepts.md)
* [파워쉘](execution/powershell/README.md)
  * [인메모리 실행](execution/powershell/in-memory-execution.md)
  * [C# 실행](execution/powershell/csharp-execution.md)
  * [윈도우 API 실행](execution/powershell/winapi-execution.md)
* [LOLBAS](execution/lolbas.md)
* [Native API - TODO](execution/native-api.md)

## 지속성 (Persistence)

* [개념](persistence/concepts.md)
* [골든 티켓 (Golden Ticket)](persistence/golden-ticket.md)
* [DLL 사이드로딩 (DLL Side-Loading)](persistence/dll-sideloading.md)
* [DLL Search Order Hijacking - TODO](persistence/dll-hijacking.md)
* [레지스트리 / 스타트업 폴더](persistence/registry-startup-folder.md)

## 권한 상승 - TODO  <a href="#privilege-escalation" id="privilege-escalation"></a>

* [개념](privilege-escalation/concepts.md)
* [로컬 권한 상승 - TODO ](privilege-escalation/local/README.md)
  * [잘못된 서비스 설정](privilege-escalation/local/misconfigured-services.md)
  * [Unquoted Service Path](privilege-escalation/local/unquoted-service-path.md)
  * [Always Install Elevated](privilege-escalation/local/always-install-elevated.md)
  * [PrintNightmare](privilege-escalation/local/printnightmare.md)
* [AD 권한 상승](privilege-escalation/ad/README.md)
  * [Active Directory Certificate Services (ADCS)](privilege-escalation/ad/adcs.md)
    * [ESC1](privilege-escalation/ad/adcs/esc1.md)
    * [ESC8](privilege-escalation/ad/adcs/esc8.md)
  * [Shadow Credentials](privilege-escalation/ad/shadow-credentials.md)
  * [noPac](privilege-escalation/ad/nopac.md)
  * [Kerberoasting](privilege-escalation/ad/kerberoasting.md)
  * [AS-REP Roasting](privilege-escalation/ad/asrep-roasting.md)

## 보안 우회 (Defense Evasion)

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

## 계정 정보 탈취 (Credential Access)

* [커버로스](credential-access/kerberos/README.md)
  * [커버로스팅 (Kerberoasting)](credential-access/kerberos/kerberoasting.md)
  * [AS-Rep Roasting](credential-access/kerberos/as-rep-roasting.md)
  * [RBCD (Resource Based Constrained Delegation) - TODO](credential-access/kerberos/rbcd.md)
* [비밀번호 스프레이 공격](credential-access/password-spraying.md)
* [LLMNR/NBT-NS 포이즈닝](credential-access/llmnr-nbtns-poisoning.md)
* [NTLM 릴레이 (NTLM Relay)](credential-access/ntlm-relay/README.md)
  * [SMB to SMB](credential-access/ntlm-relay/smb-to-smb.md)
  * [SMB to LDAP/S](credential-access/ntlm-relay/smb-to-ldap-s.md)
  * [HTTP to LDAP](credential-access/ntlm-relay/http-to-ldap.md)
  * [SMB to HTTP](credential-access/ntlm-relay/smb-to-http.md)
* [강제 인증 (Authentication Coercion)](credential-access/authentication-coercion/README.md)
  * [MS-RPRN - Printerbug / Print Spooler](credential-access/authentication-coercion/ms-rprn.md)
  * [MS-EFSRPC - Petitpotam](credential-access/authentication-coercion/ms-efsrpc.md)
  * [MS-FSRVP - ShadowCoerce](credential-access/authentication-coercion/ms-fsrvp.md)
  * [MS-DFSNM - DFSCoerce](credential-access/authentication-coercion/ms-dfsnm.md)
* [NTLM 다운그레이드](credential-access/ntlm-downgrade.md)

## 횡적 이동 (Lateral Movement)

* [개념](lateral-movement/concepts.md)
* [Pass-the-Hash](lateral-movement/pass-the-hash.md)
* [SMB 와 PsExec](lateral-movement/smb-psexec.md)
* [WMI](lateral-movement/wmi.md)
* [WinRM / Powershell Remoting](lateral-movement/winrm-ps-remoting.md)
* [RDP](lateral-movement/rdp.md)
* [Network Pivoting (피벗) - TODO](lateral-movement/pivoting.md)

## 개념 <a href="#general-concepts" id="general-concepts"></a>

* [윈도우 사용자 인증](general-concepts/windows-authentication/README.md)
  * [NTLM 인증](general-concepts/windows-authentication/ntlm.md)
  * [커버로스 (Kerberos) 인증 - TODO](general-concepts/windows-authentication/kerberos-authentication.md)
  * [ADCS 인증서 기반 인증 ](general-concepts/windows-authentication/adcs-auth.md)
* [AD 관련 용어 해설](general-concepts/glossary.md)

## 실 공격 TTP와 대응방안 - TODO <a href="#real-attack-ttp-and-mitigations" id="real-attack-ttp-and-mitigations"></a>

* [개념](real-attack-ttp-and-mitigations/concepts.md)

## 홈 랩 (Home lab) <a href="#homelab" id="homelab"></a>

* [시스몬 (sysmon) 설치](homelab/installing-sysmon.md)
* [SIEM과 EDR 솔루션 설치](homelab/edr.md)
* [취약한 랩을 위한 설정 커맨드](homelab/homelab-misconfigurations.md)

## MISC <a href="#MISC" id="MISC"></a>

* [기여하는 방법](MISC/contributions.md)
* [레퍼런스와 크레딧](MISC/레퍼런스-크레딧.md)
* [C# snippets](MISC/csharp-snippets.md)
* [winapi 리스트](MISC/winapi-list/README.md)
  * [original notes from obsidian](MISC/winapi-list/original-notes-from-obsidian.md)
* [Future Works and Research](MISC/future-works-and-research.md)
* [파워쉘 원라이너 (oneliner)](MISC/powershell-oneliners.md)
