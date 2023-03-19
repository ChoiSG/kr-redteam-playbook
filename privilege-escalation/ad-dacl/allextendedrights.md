# AllExtendedRights

* 액티브 디렉토리의 다양한 Extended Rights를 사용할 수 있는 권한. Create/Delete Child, Read/Write Property, Control Access, Take Ownership, Write Owner 등이 있다. 특별 권한을 이용해 다양한 공격을 진행해 대상을 장악할 수 있다.
* 블러드하운드 Edge

### 악용 종류&#x20;

* 유저: ForceChangePassword 권한으로 비밀번호 강제 변경
* 컴퓨터: LAPS를 이용해 `ms-MCS-AdmPwd` 특성값에 있는 로컬 관리자의 비밀번호 획득
* 도메인: `DS-Replication-Get-Changes`, `DS-Replication-Get-Changes-All` Extended Rights를 사용해 DCSync 진행



각 공격 들은 [rbcd.md](../ad/rbcd.md "mention") , [laps-todo.md](../../credential-access/laps-todo.md "mention"), [dcsync.md](../../credential-access/dcsync.md "mention") 등의 페이지를 참고한다.&#x20;

