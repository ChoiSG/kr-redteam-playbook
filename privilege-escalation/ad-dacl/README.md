# AD-DACL

이 페이지에서는 액티브 디렉토리내의 다양한 접근제어(Access Control)와 관련된 개념들을 아주 간단하게 알아본 뒤, 각 접근통제 권한을 어떻게 악용할 수 있는지에 대해서 알아본다. 이 프로젝트는 오퍼레이터들을 위한 플레이북이지 기본 보안 개념서가 아니기에, 필요한 개념에 관련된 설명은 레퍼런스 섹션을 참고한다.

* ACE (Access Control Entries, 액세스 제어 항목): ACE는 특정한 Security Principal (유저, 그룹, 컴퓨터, 등)의 접근 권한을 부여, 또는 거부하는 특정한 권한이다. ACE는 접근 권한이 부여되는 대상, 권한을 허용/거부 받는 주체를 식별할 수 있는 보안 식별자 (SID), 그리고 액세스 마스크로 이뤄져있다.
* ACL (Access Control List, 액세스 제어 리스트): ACL은 ACE의 집합이다. 개별 ACE를 적용하기엔 ACE 갯수가 너무 많아 ACL이라는 집합을 만든 뒤, 이를 주체와 대상에게 부여하는 경우가 많다.
* DACL (Discretionary Access Control Lists, 임의적 사용자 기반 액세스 제어 리스트): ACL의 종류 중 하나. 객체 (Object)의 접근을 제어하는데 사용되는 윈도우 액티브 디렉토리 ACL 의 종류다. 특정 객체에 대해 허용되거나 거부되는 보안 주체와, 그 접근 유형을 갖고 있는 ACL이다.

간단하게 정리하자면 윈도우 액티브 디렉토리에서 액세스/접근 제어를 위해서는 DACL 이 사용되며, 각각의 항목은 ACE로 이뤄져있다.

### 모의해커를 위한 DACL, ACE

내부망 모의해킹 진행시 액티브 디렉토리내의 잘못 설정된 DACL 및 ACE 들이 있다면, 공격자들은 이 접근 제한 항목/리스트 들을 이용해 다양한 공격을 진행할 수 있게 된다.

DACL/ACE를 악용할 때 신경써야하는 것들은 다음과 같다.

1. DACL/ACE를 가지고 있는 주체가 무엇인가
2. DACL/ACE가 적용되는 대상의 종류 - 유저, 컴퓨터, 그룹, OU
3. DACL/ACE 악용을 할 때의 작전 보안 및 주의할 점

### 주의점

이 페이지에서는 블러드하운드 Edge 와 액티브 디렉토리의 권한에 대해서 알아본다. 예를들어 `AddAllowedToAct` 는 공식 액티브 디렉토리 권한이 아니라, 블러드하운드에서 임의로 만든 Edge 이름이다. 그러나 GenericAll 은 공식 액티브 디렉토리 권한임과 동시에 블러드하운드의 Edge 이름이기도 하다. 이처럼 둘이 섞여있어 헷갈릴 수 있으니, 설명란에다가 표시 해놨다.

### DACL Summary

* 레퍼런스: https://www.thehacker.recipes/ad/movement/dacl&#x20;

| Name                           | Type            | Description                                                                                      | Abuse                                                                                                                                                                                                                                                                                         |
| ------------------------------ | --------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GenericAll                     | Access Right    | Combination of all other rights                                                                  | <p>- User: ShadowCreds, TargetedKerberoast, ForceChangePassword<br>- Machine: RBCD, ShadowCreds, LAPS<br>- Group: AddMember<br>- GPO: Evil GPO (Dangerous)<br>- Domain: DCSync<br>- OU/Container: DaclEdit + FullControl + ShadowCreds</p>                                                    |
| GenericWrite                   | Access Right    | Combination of write permissions                                                                 | <p>- User: ShadowCreds, TargetedKerberoast<br>- Machine: RBCD, ShadowCreds<br>- Group: AddMembers<br>- GPO: Evil GPO (Dangerous)</p>                                                                                                                                                          |
| AllExtendedRight               | Access Right    | Extended Rights like Create/Delete Child, Read/Write Property, Control Access, Write Owner, etc. | <p>- User: ForceChangePassword<br>- Machine: LAPS<br>- Group: AddMember<br>- Domain: DCSync, LAPS</p>                                                                                                                                                                                         |
| WriteDACL                      | Access Right    | Edit object's DACL                                                                               | <p>- User: FullControl + ShadowCreds,TargetedKerberoast,ForceChangePassword<br>- Machine: FullControl + ShadowCreds, RBCD<br>- Group: WriteMembers + AddMember<br>- Domain: DCSync + DCSync<br>- GPO: Pass (Dangerous)</p>                                                                    |
| WriteOwner                     | Access Right    | Change the owner of the object + WriteDACL using ownership                                       | <p>- User: OwnerEdit + WriteDACL + FullControl + ShadowCreds, TargetedKerberoast<br>- Group: OwnerEdit + WriteDACL + WriteMember + AddMember<br>- Machine: OwnerEdit + WriteDACL + FullControl + RBCD, ShadowCreds, LAPS<br>- Domain: OwnerEdit + WriteDACL + FullControl (dangerous)<br></p> |
| AddAllowedToAct                | BloodHound Edge | Modify `msDS-AllowedToActOnBehalfOfOtherIdentity`                                                | - Machine: RBCD                                                                                                                                                                                                                                                                               |
| AddKeyCredentialLink           | BloodHound Edge | Modify `msDS-KeyCredentialLink`                                                                  | - User: ShadowCreds - Machine: ShadowCreds                                                                                                                                                                                                                                                    |
| WriteAccountRestrictions       | BloodHound Edge | Modify `msDS-AllowedToActOnBehalfOfOtherIdentity`                                                | - Machine: RBCD                                                                                                                                                                                                                                                                               |
| DS-Replication-Get-Changes     | Extended Right  | One of the two extended rights needed to do DCSync                                               | - Domain: DCSync                                                                                                                                                                                                                                                                              |
| DS-Replication-Get-Changes-All | Extended Right  | One of the two extended rights needed to do DCSync                                               | - Domain: DCSync                                                                                                                                                                                                                                                                              |

## 레퍼런스&#x20;

* https://github.com/BloodHoundAD/BloodHound/pull/625/files#diff-def72dd48888a7b1a9ad8f7df64ff1e7d5a5b664df9f47614794a9c9bf87ff26
* https://learn.microsoft.com/ko-kr/windows/win32/ad/how-access-control-works-in-active-directory-domain-services
* https://learn.microsoft.com/ko-kr/windows/win32/ad/controlling-access-to-objects-and-their-properties
* https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse
* https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
* https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab
* https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?redirectedfrom=MSDN\&view=dotnet-plat-ext-7.0
* https://dirkjanm.io/abusing-forgotten-permissions-on-precreated-computer-objects-in-active-directory/
* https://www.youtube.com/watch?v=z8thoG7gPd0
* https://www.thehacker.recipes/ad/movement/dacl
* [https://simondotsh.com/infosec/2021/12/29/dcom-without-admin.html](https://simondotsh.com/infosec/2021/12/29/dcom-without-admin.html)&#x20;
