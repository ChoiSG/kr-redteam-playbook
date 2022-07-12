# Table of contents

* [레드팀 플레이북](README.md)
* [레드팀이란](레드팀이란.md)

## 인프라 (Infrastructure) <a href="#인프라" id="인프라"></a>

* [개념](인프라/개념.md)
* [예시 인프라](인프라/예시-인프라.md)
* [팀 서버 - Sliver](인프라/팀-서버-Sliver/README.md)
  * [스테이저 (Stager) 사용](인프라/팀-서버-Sliver/스테이저-사용.md)
* [도메인 분류와 신뢰도](인프라/도메인-설정.md)
* [중립 공간 (클라우드) 설정](인프라/클라우드-플랫폼-설정.md)
* [네뷸라 (Nebula)](인프라/네뷸라.md)
* [네뷸라 설정](인프라/네뷸라-설정.md)
* [도메인과 리다이렉터 설정](인프라/도메인-리다이렉터-설정.md)

## 공개 출처 정보 수집 (OSINT) <a href="#공개출처정보수집" id="공개출처정보수집"></a>

* [개념](공개출처정보수집/개념.md)
* [작전보안 - TODO](공개출처정보수집/작전보안.md)
* [자산 정보 수집](공개출처정보수집/도메인-정보.md)
* [구글 도킹](공개출처정보수집/구글-도킹.md)

## 초기 침투 (Initial Access) <a href="#초기침투" id="초기침투"></a>

* [개념](초기침투/개념.md)
* [피싱 첨부파일](초기침투/피싱-첨부파일.md)
  * [오피스 VBA 매크로](초기침투/피싱-첨부파일/오피스-매크로.md)
  * [원격 템플렛 인젝션](초기침투/피싱-첨부파일/템플렛-인젝션.md)
  * [VBA Stomping](초기침투/피싱-첨부파일/vba-stomping.md)
  * [VBA Purging - TODO](초기침투/피싱-첨부파일/vba-purging.md)
  * [LNK](초기침투/피싱-첨부파일/lnk.md)
  * [DotNetToJS](초기침투/피싱-첨부파일/dotnettojs.md)
  * [HTA](초기침투/피싱-첨부파일/hta.md)
  * [Follina](초기침투/피싱-첨부파일/follina.md)
  * [XLM Excel 4.0 매크로](초기침투/피싱-첨부파일/XLM-Excel-40.md)
* [HTML 스머글링 (Smuggling)](초기침투/html-smuggling.md)
* [피싱 - AitM (Adversary in the Middle)](초기침투/피싱-aitm.md)
* [Living Off Trusted Sites (LOTS)](초기침투/living-off-trusted-sites.md)
* [비밀번호 스프레이 공격 - TODO ](초기침투/비밀번호-스프레이-공격.md)

## 정보 수집 - 내부망 <a href="#정보수집" id="정보수집"></a>

* [개념](정보수집/개념.md)
* [로컬 호스트 정보 수집](정보수집/로컬-호스트-정보-수집.md)
* [블러드하운드](정보수집/블러드하운드.md)
* [SMB 쉐어 수집](정보수집/smb-share.md)
* [정보 수집 - 파워쉘 - TODO](정보수집/정보-수집-파워쉘.md)
* [정보 수집 - C# - TODO](정보수집/정보-수집-시샵.md)

## 실행 (Execution) <a href="#실행" id="실행"></a>

* [개념](실행/개념.md)
* [파워쉘](실행/파워쉘/README.md)
  * [인메모리 실행](실행/파워쉘/인메모리-실행.md)
  * [C# 실행](실행/파워쉘/시샵-실행.md)
  * [윈도우 API 실행](실행/파워쉘/윈도우-API-실행.md)
* [LOLBAS](실행/lolbas.md)
* [Native API - TODO](실행/native-api.md)

## 지속성 (Persistence) <a href="#지속성-공격" id="지속성-공격"></a>

* [개념](지속성-공격/개념.md)
* [골든 티켓 (Golden Ticket)](지속성-공격/골든-티켓.md)
* [DLL 사이드로딩 (DLL Side-Loading)](지속성-공격/dll-사이드로딩.md)
* [DLL Search Order Hijacking - TODO](지속성-공격/dll-hijacking.md)
* [스타트업 폴더 - TODO](지속성-공격/스타트업-폴더.md)

## 권한 상승 - TODO  <a href="#권한-상승" id="권한-상승"></a>

* [개념](권한-상승/개념.md)
* [로컬 권한 상승 - TODO ](권한-상승/로컬-권한-상승/README.md)
  * [잘못된 서비스 설정](권한-상승/로컬-권한-상승/잘못된-서비스-설정.md)
  * [Unquoted Service Path](권한-상승/로컬-권한-상승/unquoted-service-path.md)
  * [Always Install Elevated](권한-상승/로컬-권한-상승/always-install-elevated.md)
  * [PrintNightmare](권한-상승/로컬-권한-상승/printnightmare.md)
* [AD 권한 상승 - TODO ](권한-상승/ad-권한-상승/README.md)
  * [Active Directory Certificate Services (ADCS)](권한-상승/ad-권한-상승/active-directory-certificate-services-adcs.md)
  * [noPac](권한-상승/ad-권한-상승/nopac.md)
  * [Kerberoasting](권한-상승/ad-권한-상승/kerberoasting.md)

## 보안 우회 (Defense Evasion)

* [AMSI 우회](defense-evasion/amsi-우회.md)
* [유저랜드 후킹 - 역사](defense-evasion/유저랜드-후킹-역사.md)
* [유저랜드 커널랜드 윈도우API 개념](defense-evasion/유저-랜드-후킹-개념.md)
* [유저랜드 후킹](defense-evasion/유저랜드-후킹.md)
* [DInvoke - 시스템 콜](defense-evasion/dinvoke-syscall.md)
* [페이로드 크기](defense-evasion/페이로드-크기.md)
* [가변적 C2 프로필](defense-evasion/malleable-c2-profile.md)
* [프로세스 인젝션](defense-evasion/프로세스-인젝션/README.md)
  * [CreateRemoteThread](defense-evasion/프로세스-인젝션/createremotethread.md)
  * [NtMapViewOfSection](defense-evasion/프로세스-인젝션/ntmapviewofsection.md)
* [간단 디펜더 우회](defense-evasion/간단-디펜더-우회.md)

## 계정 정보 탈취 (Credential Access) <a href="#계정-정보-탈취" id="계정-정보-탈취"></a>

* [커버로스](계정-정보-탈취/커버로스/README.md)
  * [커버로스팅 (Kerberoasting)](계정-정보-탈취/커버로스/커버로스팅.md)
  * [AS-Rep Roasting](계정-정보-탈취/커버로스/as-rep-roasting.md)
* [비밀번호 스프레이 공격](계정-정보-탈취/비밀번호-스프레이-공격.md)
* [LLMNR/NBT-NS 포이즈닝](계정-정보-탈취/llmnr-nbtns-poisoning.md)
* [강제 인증 (Authentication Coercion)](계정-정보-탈취/강제-인증/README.md)
  * [MS-RPRN - 프린터 버그 / 프린트 스풀러 ](계정-정보-탈취/강제-인증/ms-rprn.md)
  * [MS-EFSRPC - Petitpotam](계정-정보-탈취/강제-인증/ms-efsrpc.md)
  * [MS-FSRVP - ShadowCoerce](계정-정보-탈취/강제-인증/ms-fsrvp.md)
  * [MS-DFSNM - DFSCoerce](계정-정보-탈취/강제-인증/ms-dfsnm.md)
* [NTLM 릴레이 - TODO ](계정-정보-탈취/ntlm-릴레이.md)
  * [SMB to SMB](계정-정보-탈취/ntlm-릴레이/smb-to-smb.md)
* [NTLM 다운그레이드](계정-정보-탈취/ntlm-다운그레이드.md)
* [RBCD (Resource Based Constrained Delegation) - TODO](계정-정보-탈취/rbcd.md)

## 횡적 이동 (Lateral Movement) <a href="#횡적-이동" id="횡적-이동"></a>

* [개념](횡적-이동/개념.md)
* [Pass-the-Hash](횡적-이동/pass-the-hash.md)
* [SMB 와 PsExec](횡적-이동/smb-psexec.md)
* [WMI](횡적-이동/wmi.md)
* [WinRM / Powershell Remoting](횡적-이동/winrm-ps-remoting.md)
* [RDP](횡적-이동/rdp.md)

## 홈 랩 구축 <a href="#홈-랩-구축" id="홈-랩-구축"></a>

* [시스몬 (sysmon) 설치](홈-랩-구축/시스몬-설치.md)

## 실 공격 TTP와 대응방안 - TODO <a href="#real-attack-ttp-and-mitigations" id="real-attack-ttp-and-mitigations"></a>

* [개념](real-attack-ttp-and-mitigations/개념.md)

## 개념 <a href="#개념" id="개념"></a>

* [윈도우 사용자 인증](개념/윈도우-사용자-인증/README.md)
  * [NTLM 인증](개념/윈도우-사용자-인증/ntlm.md)
  * [커버로스 (Kerberos) 인증 - TODO](개념/윈도우-사용자-인증/kerberos-authentication.md)
  * [ADCS 인증서 기반 인증 ](개념/윈도우-사용자-인증/adcs-auth.md)

## MISC <a href="#MISC" id="MISC"></a>

* [기여하는 방법](MISC/기여하는-방법.md)
* [C# snippets](MISC/csharp-snippets.md)
* [winapi 리스트](MISC/winapi-리스트/README.md)
  * [original notes from obsidian](MISC/winapi-리스트/original-notes-from-obsidian.md)
* [Future Works and Research](MISC/future-works-and-research.md)
* [레퍼런스와 크레딧](MISC/레퍼런스-크레딧.md)
* [파워쉘 원라이너 (oneliner)](MISC/파워쉘-원라이너.md)
