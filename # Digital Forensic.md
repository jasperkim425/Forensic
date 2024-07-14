# Digital Forensic

## 디스크 쓰기 방지 설정
- Win + R 실행
- regedit
- HKEY_Local_Machine\SYSTEM\CurrentControlSet\Control 키 새로만들기
- StorageDevicePolicies 생성
- DWORD(32비트) 새로생성
- 값 이름 WriteProtect로 변경
- 데이터 값 1로 변경
- 변경 후 USB 연결

## 사본 이미지 생성
- FTK Imager - File - Create Disk Image - Physical Drive - USB 선택 - Verify images after they are created 체크 해제 - Add - Raw(dd) - 다음 - 폴더 선택 및 이름 설정 - Image Fragment Size 0으로 설정 
- 이미지파일과 함께 생성되는 txt 파일은 이미지 정보 저장(모델, 시리얼 넘버, 해시값)

## 파일시스템 복구
- 모든 파일시스템을 FTK Imager로 열어 훼손되었는지 확인 후 파일시스템 종류 확인
- HxD 에서 도구 - 디스크 이미지 열기로 디스크 오픈하면 섹터 단위로 이동 가능
- 복사한 VBR 정보는 ctrl + B를 사용하여 덮어쓰기
    - FAT32 : 섹터 6으로 이동하여 VBR 복사본 복사하여 덮어쓰기로 복구
    - NTFS : 섹터의 마지막으로 이동하여 VBR 복사본 복사하여 덮어쓰기로 복구
    - NTFS VBR 상단만 지워진 경우 : 찾기에 ntfs를 입력하여 검색한 후 EB52904E544653 hex 값이 나오면 복사하여 덮어쓰기로 복구
    - 다중 파티션
        1) 훼손된 파티션들의 시작 섹터를 FTK Imager를 이용하여 확인
        2) 또한 FTK Imager에서 파일시스템도 확인
        3) 그리고 FAT32는 시작 섹터 + 6, NTFS는 파티션의 마지막으로 이동하여 복사
        4) 복사한 VBR 복사본을 각 시작 섹터에 덮어쓰기

## 총 섹터 수
- MBR이 있는 FAT32 : 파티션 타입 확인 (파티션 테이블의 5번째 값 : 0x0B), 파티션 테이블의 마지막 4byte
- MBR이 없는 FAT32 : VBR 복사본에서 3번째 줄 처음 4byte
- MBR이 있는 NTFS : 파티션 타입 확인 (파티션 테이블의 5번째 값 : 0x07), 파티션 테이블의 마지막 4byte
- MBR이 없는 NTFS : VBR 복사본에서 3번째 줄 마지막 8byte

## USB 분석
- USB 시리얼 넘버 
    1) FTK Imager - File - Add Evidence Item - Physical Drive - USB - 왼쪽 하단 Properties 에서 확인 가능
    2) 이미지 생성 시 생성되는 txt 값에서 확인 가능
- USB 내 볼륨 시리얼 넘버
    1) FTK Image - 볼륨 클릭 - Properties에서 확인 가능
    2) 풀 시리얼(빅에디안) : 볼륨 클릭 후 offset 0x48-0x4F(리틀 에디안으로 표시) - 빅에디안으로 변경

## 해시값 확인
- 윈도우 터미널 - `certutil -hashfile [파일명] [해시알고리즘]`

## 바로가기 파일(.ink) 분석
- .ink 파일의 정보를 알기 위해 LNK Parser 설치 (http://forensic.korea.ac.kr/tools.html)
- 파일 이름, 파일 위치, 연결된 볼륨 시리얼 넘버 등 확인 가능
- 볼륨 시리얼 넘버 확인은 FTK Imager에서 매칭되는 시리얼 넘버 확인
- 볼륨 시리얼 넘버 확인 후 autopsy에서 해당 볼륨에서 파일 탐색
- 만약 파일이 존재 하지 않는다면 $MFT, $LogFile 추출하여 NTFS Log Tracker 실행 후 알맞게 삽입
- Parse 버튼 클릭 후 결과를 저장할 SQLite DB File 이름과 저장할 위치 선택
- 존재하지 않는 파일명 검색

## 레지스트리 분석
- 파일시스템 생성된 날짜 확인(autopsy) : 디스크 이미지 - 복구된 드라이브 - $MFT 파일 생성시간으로 확인 가능
- 운영체제 설치 날짜(autopsy) : Windows - system32 - config -
- 네트워크 GUID 확인(autopsy) : Windows - system32 - config - SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkCards\
- 네트워크 연결정보(autopsy) : Windows - System32 - config - SYSTEM\ControlSet001\Services\Tcpip\Parameters\Interfaces\{GUID}
- 네트워크 MAC 주소(autopsy) : Windows - System32 - config - SYSTEM\controlSet001\Control\NetworkSetup2\Interfaces\{GUID}\Kernel - CurrentAddress
- 연결된 네트워크 공유기 정보(autopsy) : Windows - system32 - config - SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles\{GUID} 
    - DateCreated : 네트워크 처음 연결된 날짜
    - DateLastConnected : 마지막으로 연결된 날짜
- 네트워크 공유기 MAC 주소 : Windows - system32 - config - SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\
    - 위 Profiles의 GUID에서 Managed 값 확인
        - Managed 값이 0일 경우 : Unmanaged 키 참조
        - Managed 값이 1일 경우 : Managed 키 참조

## 안티포렌식

- 데이터 삭제기법 : Eraser, SDelete, Drive Cleanser
- 암호화 및 변조(난독화) : BitLocker, LockMyPix, Truecrypt
- 데이터 은닉 : PE파일 헤더 제거, Bootice, 플래시 메모리
- USB 은닉 : FbinstTool




## 비트코인 지갑




## 가상자산

## 가상머신

## 참조
- 