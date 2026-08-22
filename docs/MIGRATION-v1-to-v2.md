# v1 → v2 메모

아직 이전 Multi-Profile Egg를 실제 서버에 적용하지 않았다면 **v2만 사용하면 됩니다.**

v2는 다음을 추가했습니다.

- 서버 종류 자동 감지
- Minecraft 버전 탐지
- Java 8/11/16/17/21/25 자동 선택 및 캐시
- 프로필별 `profile.conf`
- 잘못된 프로필/JAR 실행 방지
- 프로필 전환 시 이전 프로필 자동 백업

기존 서버 파일을 가져올 때는 각 서버를:

```text
/home/container/servers/<프로필명>/
```

아래에 통째로 넣으면 됩니다.

오래된 서버이고 JAR 이름이 단순히 `server.jar`라 자동 버전 감지가 안 될 경우,
해당 폴더의 `profile.conf`에 한 줄만 지정하면 됩니다.

```ini
MINECRAFT_VERSION=1.12.2
```

Java를 직접 고정하고 싶다면:

```ini
JAVA_VERSION=8
```

처럼 지정할 수도 있습니다.
