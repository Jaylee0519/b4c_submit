## 도커(docker) 구축
도커 : 리눅스 컨테이너 기반으로 하는 오픈소스 가상화 플랫폼, 특정 서비스를 패키징하고 배포하는데 유용한 오픈소스 프로그램.
#### 도커를 사용한 서버 구축 이유
- 클라우드 서버환경과 동일한 개발환경을 구축할 수 있음.
- 협업 시 모든 인원이 로컬 네트워크 환경에 있는 것은 아님 => 팀원들의 개발 환경 동일성을 위해 도커 이미지를 배포, 도커 컨테이너 이용.
- 애플리케이션 독립성을 가짐 => 호스트OS, 다른 컨테이너와 독립적인 공간을 보장. 충돌 발생X 

### Docker Compose 파일

```
name: wordpress
services:
	wordpress:
		image: wordpress:latest
		container_name: wordpress-web
		environment:
			WORDPRESS_DB_HOST: wordpress-db:3306
			WORDPRESS_DB_USER: root
			WORDPRESS_DB_PASSWORD: wordpress
		ports:
			- 8080:80
		depends_on:
			- mysql
	mysql:
		image: mysql:latest
		container_name: wordpress-db
		environment:
			MYSQL_ROOT_PASSWORD: wordpress
			MYSQL_DATABASE: wordpress
			MYSQL_PASSWORD: wordpress
```
: 해당 코드는 WordPress와 MySQL을 함께 구성하는 환경을 정의한 파일.
=> WordPress사이트와 데이터베이스를 손쉽게 Docker 컨테이너로 구동 가능함.

#### 파일 구조
- `wordpress`와 `mysql` 두 가지 서비스로 나누어져 있으며, 각 서비스는 개별 컨테이너로 구동되어짐.
- 해당 파일을 통해 `docker-compose up` 명령어로 실행 => 두 컨테이너가 동시에 실행, 서로 네트워크로 연결됨.
#### 각 서비스의 역할 및 설정
1. **WordPress 서비스**
	- **이미지** : `image: wordpress: latest` 
		✔️ `wordpress : latest` 이미지를 기반으로 컨테이너 생성 
		(※ WordPress 애플리케이션이 포함된 표준 Docker 이미지를 사용함 )
		
	- **컨테이너 이름** : `container_name: wordpress-web`
		✔️생성된 컨테이너 이름을 `wordpress-web`으로 지정 (구분용)
		
	- **환경변수** :
		✔️ `WORDPRESS_DB_HOST` : db 호스트의 주소 => WordPress가 연결할 MySQL db를`wordpress-db:3306` 주소로 지정.
		✔️ `WORDPRESS_DB_USER` 및 `WORDPRESS_DB_PASSWORD`:  WordPress가 사용할 db 사용자명과 비밀번호를 지정함.
		
	- **포트 매핑** : `ports: - 8080:80`
		✔️로컬머신의 8080포트, WordPress 컨테이너의 80포트 연결 =>`http://localhost:8080`로 WordPress 사이트에 접근
		
	- **종속성 설정** : `depends_on: - mysql`
		✔️MySQL 데이터베이스가 먼저 구동되도록 함.
	
2. **MySQL 서비스**
	- **이미지** : `image: mysql:latest`
	    ✔️ `mysql:latest` 이미지를 사용하여 데이터베이스 컨테이너를 생성.
	    
	- **컨테이너 이름**: `container_name: wordpress-db`
	    ✔️ 생성된 컨테이너 이름을 `wordpress-db`로 지정 => 다른 db 컨테이너와 구분.
	    
	- **환경 변수**:
	    ✔️`MYSQL_ROOT_PASSWORD`: MySQL의 루트 비밀번호를 설정.
	    ✔️`MYSQL_DATABASE`: WordPress가 사용할 db 이름을 `wordpress`로 생성.
	    ✔️`MYSQL_PASSWORD`: WordPress가 사용할 db 사용자 비밀번호를 설정.

### Docker Compose 구동 과정
환경 구성 => 컨테이너 생성 및 연결 => 환경변수 설정 적용 => 웹 서버 접근

**vs code에서 실행한 Docker Compose 파일**
![[스크린샷 2024-11-06 오전 1.12.35.png]]※ `.yml`과 `.yaml` 확장자 둘 다 YAML파일을 나타내며 기능적인 차이가 없음.



## WordPress란?
: 콘텐츠 관리 시스템 소프트웨어(CMS)의 일종으로, 전문적 지식 없이도 웹사이트에서 콘텐츠를 생성, 관리 및 수정할 수 있도록 해줌. 아래와 같이 편리한 사용번과 확장성으로 수많은 대기업에서 활용한다고 함.
![[스크린샷 2024-11-06 오전 1.30.03.png]]
#### WordPress의 장점
- **무료 오픈소스**
	: 수많은 **플러그인** 제작자들에 의해 제공되는 무료(유료) 플로그인 설치
- **확장성**
	: 현재 5만개 이상의 무료 플러그인, 5천개 이상의 테마 존재 => 웹사이트 수정 및 기능 추가에 용이



### 다운그레이드와 원데이 취약점 연구
: 플러그인을 다운그레이드 하면 과거 버전의 수정되지 않은 취약점 (**1-day 취약점**) 분석 가능성up

 **원데이 취약점**이란?
 : 보안 커뮤니티나 대중에게 이미 알려진 취약점, 패치가 배포되었으나 **과거 버전에는 여전히 남아있음.**

- 취약점이 수정되지 않은 버전 확인하기
- 패치와 취약점 비교 : 다운그레이드 후 원데이 취약점을 통해 어떤 부분이 변경되었는지, 패치가 어떻게 적용되었는지 비교 가능.


### WordPress 플러그인 다운그레이드 방법 

1. **WP Rollback 플러그인** (관리자 페이지에 로그인이 가능한 경우)
	: 워드프레스 알림판 => 플러그인 => 새로추가 페이지 : wp rollback 설치 및 활성화
	=>  개별 플러그인의 버전을 과거버전으로 롤백할 수 있는 **Rollback**링크가 표시됨.
	
2. **워드프레스 플러그인 페이지에서 이전 버전 다운로드하기** (🌟)
	a.) 플러그인 페이지 방문
	b.) 이전 플러그인 버전 파일 다운로드 
		: **Advanced view** 클릭 => **Previous version** 섹션 => **Development version** 드롭다운 클릭 => 버전 선택 및 다운받기.

예시>
![[스크린샷 2024-11-06 오전 1.50.52.png]]


### 플러그인 파일 업로드

 1. 관리자 대시보드에서 업로드
	 - 설치된 플러그인 `.zip` 파일 준비 => 워드프레스 대시보드로 이동
	 - 플러그인 메뉴 : **플러그인> 새로추가**
	 - **플러그인 업로드** ... `.zip`파일 선택 => **플러그인 활성화**
	 
 2. FTP 클라이언트를 이용한 업로드
	 - FTP클라이언트 사용 => 호스팅 서버 접속
	 - 플러그인 폴더로 이동 : `wp-content/plugins/` 
	 - '로컬에서' 플러그인 압축파일 해제 => `plugins` 디렉토리 내에 업로드
	 - 워드프레스 관리자화면 (**플러그인 메뉴 > 활성화**)
	 
 3. **`docker cp`** 를 통한 업로드
	 - 설치된 플러그인 `.zip` 파일 준비 (아래의 경우, `classic-editor.1.6.4.zip` 파일)
	 - **`docker ps`** 명령 => 플러그인이 설치될 WordPress 컨테이너 이름을 확인함
	 - **`docker cp`** 명령 => 설치할 `plugins` 디렉토리로 파일을 복사
	 - **컨테이너 내부**에서 파일 압축 해제하기 : **`docker exec -it wordpress-web bash`**
	 - **플러그인 압축 해제** 및 **활성화**

**터미널 창 모습**
![[스크린샷 2024-11-06 오전 1.08.26.png]]



## CVE-2023-35878
: 해당 취약점은 WordPress의 Extra User Details 플러그인 (**0.5버전 이하**) 에서 발생한 **Stored Cross-Site-Scripting(XSS)**

```
<input 
    name="eud_fields[' . esc_attr( $key ) . '][2]" 
    type="text" 
    value="' . ( 
        isset( $value[2] ) 
            ? htmlspecialchars( stripslashes( $value[2] ), ENT_NOQUOTES ) 
            : '' 
    ) . '" 
    class="regular-text" 
    size="80" 
/>
```
위 코드에서 사용자 입력을 HTML `<input>` 필드에 삽입하는 과정에서 발생하는 취약점이 있음.

- **`htmlspecialchars()`** : 특수문자를 HTML Entity로 변환하지만, **`ENT_NOQUOTES`** 옵션이 적용되어 있어 `'`와 `"` 같은 따옴표는 변환되지 않음.
  => 공격자가 따옴표를 이용해 XSS 페이로드를 삽입할 수 있는 가능성.
  
- **`stripslashes()`**: 백슬래시를 제거해주는 함수 => 데이터가 이스케이프된 경우, 이를 복원하는 용도로 사용.  ( 해당 함수는 입력값을 처리할 때 XSS를 방어하는 기능이 없음. )

- **`ENT_NOQUOTES`** 옵션 : `htmlspecialchars()`가 `<`,`>`는 변환하지만 `"`,`'`는 변환되지 않음.
 
※ XSS방어의 핵심은 HTML 내의 특수문자 ( `<`, `>`, `"`, `'` ) Entity로 변환해 브라우저가 이를 코드로 해석하지 못하도록 하는 것!


### XSS 방어를 위한 함수
1. `htmlspecialchars()` : PHP에서 주로 사용되는 함수 => 입력된 문자열에서 특수 문자를 HTML 엔터티로 변환.
2. **`esc_attr()`** : 워드프레스에서 HTML 속성에 사용할 때, 안전하게 문자열을 처리해줌.
3. `esc_html()` : HTML 본문 내에서 사용할 때, 안전하게 문자열을 처리해줌. etc)
... 위의 경우, 해당 코드 부분에만 **`esc_attr()`** 함수가 존재X


### 공격 시나리오 및 페이로드

공격자가 아래와 같은 페이로드를 입력한다고 가정.
```
Payload : " onclick=alert(document.cookie) "
```

- `onfocus`라는 JavaScript 이벤트 핸들러를 포함.
- `ENT_NOQUOTES` 플래그가 적용된 상태에서 큰따옴표(`"`)가 변환되지 않음
  => 입력 필드의 속성 값으로 그대로 들어가게 되며, 악성 스크립트가 HTML 속성으로 삽입됨. 
  => 필드에 초점을 맞추면 `alert(1)`이라는 JavaScript 코드가 실행됨.
  
**∴** 사용자가 해당 필드에 접근할 떄마다 XSS코드가 실행되므로, 악성 스크립트가 WordPress관리 페이지에서 실행되는 위험한 상황 발생.


### Solution
- `ENT_NOQUOTES` 대신 `ENT_QUOTES` 옵션을 사용 => `'`,`"` 를 HTML 엔터티로 변환
- `esc_attr()`와 같은 워드프레스 보안 함수를 사용하기

