# termux-tailscale MCP 서버 README

## 개요

이 MCP 서버는 안드로이드 폰의 Termux에서 실행되며, Tailscale을 통해 Claude 커스텀 커넥터로 연결됩니다. 도구는 크게 두 갈래입니다.

1. **Termux 도구**: 서버가 Termux 쉘과 termux-api로 직접 처리 (파일, 쉘, 패키지, 센서, SMS 등)
2. **써니폰 컨트롤 도구(phone_*)**: 서버가 명령을 받아 써니폰 컨트롤(Sunny Phone Control) 앱에 전달하고, 앱이 접근성 서비스·알림 리스너·미디어 세션으로 실제 화면 조작을 실행

## 동작 구조

```
Claude → (Tailscale, ***) → Termux MCP 서버
    ├─ Termux 도구 → Termux 쉘 / termux-api
    └─ phone_* 도구 → 써니폰 컨트롤 앱 → 접근성 서비스로 탭·입력·스크롤
```

써니폰 컨트롤 앱은 이 경로에서 MCP 서버가 아니라 **명령 실행기** 역할입니다. 별도 Android MCP 서버(Android Remote Control MCP)는 독립 관리되며 이 서버와 무관합니다.

## 1. Termux 도구

### 쉘·프로세스
- `shell`: 명령 실행 후 stdout, stderr, 종료 코드 반환
- `shell_bg`: 백그라운드 명령 시작, 작업 ID 반환
- `shell_jobs`: 백그라운드 작업 목록
- `shell_logs`: 백그라운드 작업 로그 끝부분 조회
- `shell_kill`: 백그라운드 작업 프로세스 그룹에 시그널 전송
- `proc_list`: 프로세스 목록
- `proc_kill`: 프로세스에 시그널 전송

### tmux
- `tmux_open`: 분리된 tmux 세션 생성
- `tmux_send`: 세션에 명령 입력 후 엔터
- `tmux_capture`: 세션 화면 내용 조회
- `tmux_list`: 세션 목록
- `tmux_kill`: 세션 종료

### 파일 시스템
- `fs_list`, `fs_stat`: 디렉터리 목록, 메타데이터
- `fs_read`, `fs_write`: 파일 읽기(텍스트/base64), 쓰기
- `fs_mkdir`, `fs_move`, `fs_delete`: 생성, 이동, 삭제
- `fs_search`: 문자열·정규식 재귀 검색
- `fs_download`: URL을 받아 파일로 저장
- `fs_tar`: 대량 파일 전송용 묶기

### 패키지
- `pkg_list`, `pkg_install`, `pkg_upgrade`

### 기기 정보·센서 (termux-api)
- `system_info`, `termux_info`, `tools_status`, `storage_info`
- `network_info`, `wifi_scan`, `telephony_info`
- `location`: GPS·네트워크 위치
- `camera_photo`, `mic_record`, `screenshot`, `media_list`

### 통신·연락처
- `sms_list`, `sms_send`, `call_log`, `contact_list`

### 알림·UI 보조
- `notification`, `toast`, `dialog`
- `share`: 안드로이드 공유 시트
- `open_url`: URL 열기 또는 패키지 실행
- `api_run`: 설치된 termux-* 도우미 직접 실행
- `audit_tail`: 서버 감사 로그 조회

## 2. 써니폰 컨트롤 도구 (phone_*)

### 상태
- `phone_status`: 앱 상태(접근성 서비스, 알림 리스너 등)

### 화면 읽기
- `phone_screen_text`: 현재 앱과 보이는 글자 간단 조회
- `phone_get_screen_state`: 앱 정보, 화면 크기, UI 트리
- `phone_find_nodes`, `phone_get_node_details`: 노드 검색, 상세
- `phone_wait_for_node`, `phone_wait_for_idle`: 노드 등장·화면 안정 대기

### 터치·제스처
- `phone_tap`, `phone_double_tap`, `phone_long_press`: 좌표 기반
- `phone_tap_node`, `phone_click_node`, `phone_long_click_node`: 노드 기반
- `phone_swipe`, `phone_scroll`, `phone_scroll_node`, `phone_scroll_to_node`
- `phone_pinch`, `phone_custom_gesture`

### 텍스트 입력
- `phone_fill_text`: 입력칸 찾아 글자 입력(엔터 옵션)
- `phone_type_append_text`, `phone_type_insert_text`, `phone_type_replace_text`, `phone_type_clear_text`
- `phone_press_key`: ENTER, BACK, DEL 등
- `phone_dismiss_keyboard`
- `phone_get_clipboard`, `phone_set_clipboard`, `phone_copy_text`

### 내비게이션·앱
- `phone_press_back`, `phone_press_home`, `phone_press_recents`
- `phone_open_notifications`, `phone_open_quick_settings`
- `phone_list_apps`, `phone_open_app`, `phone_close_app`
- `phone_open_uri`, `phone_send_intent`

### 알림
- `phone_notification_list`, `phone_notification_open`, `phone_notification_action`
- `phone_notification_reply`, `phone_notification_snooze`, `phone_notification_dismiss`

### 미디어
- `phone_media_status`: 재생 상태·위치·제목
- `phone_media_control`: 재생, 일시정지, 다음, 이전, 탐색
- `phone_youtube_play`: 유튜브 검색·열기·미디어 세션 재생

### 카메라·위치·공유
- `phone_list_cameras`, `phone_list_camera_photo_resolutions`, `phone_list_camera_video_resolutions`
- `phone_take_camera_photo`, `phone_save_camera_photo`, `phone_save_camera_video`
- `phone_get_location`
- `phone_get_shared_content`: 공유받은 항목 조회
- `phone_share_file_via_web`: 기기 파일을 임시 URL로 노출

## 운영 규칙

- Termux 앱에 명령 입력 시 엔터를 두 번 누름
- 알림 정리 시 MCP 서버 관련 알림은 지우지 않음
- 유튜브 재생은 화면 조작 없이 미디어 세션 제어와 재생 시간 확인으로만 처리
- 명령 전달 성공과 화면에서 확인한 결과를 구분
- 주소, 토큰, 내부 IP 등 민감한 값은 문서에서 ***로 가림
