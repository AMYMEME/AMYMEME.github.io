---
layout: post
title: 픽미업 창업 프로젝트
excerpt: "Java, Spring Boot, JPA, Criteria, EC2, RDS, S3"
categories: [단체 프로젝트]
comments: true
share : false
---

# 시작
우테코 최종에서 떨어진 후에 SW 창업경진대회 우수상을 수상했던 
대학생을 위한 프로젝트/창업 팀원 매칭 플랫폼 팀에서 백엔드를 구한다고 하길래 지원했다.  
스프링 부트로 간단한 CRUD 및 페이지네이션, 여러 조건 조합의 검색쿼리를 처리 해야 했고 AWS로 배포까지
진행하였다. 사이트는 [여기](https://pickmeup.site)에서 확인할 수 있고, API 문서는 [git book](https://chorom-m-ham.gitbook.io/pickmeup-api/)에서 확인할 수 있다. 
{: .notice}

* * *


# 과정
> 2021년 1월부터 2021년 2월 말까지 진행하였는데, 오랜만에 팀 프로젝트였고, 백엔드가 두 명이어서 코드리뷰를 할 수 있어서
> 신이 났다. 오랜만에 스프링 부트로 해보기도 했고, 디비 설계부터 할 수 있어서 더 재밌었던 것 같다. 
> 코로나 때문에 팀원들을 직접 한 번도 보지 못한 채로 끝나서 아쉬웠지만, 좀 더 깊이있게 스프링 부트를 할 수 있었던 계기가 된 것 같다.  


## 협업
### GitHub
일단 브랜치 전략은 각자 자신의 브랜치를 띄우는 `Git Flow`를 기반으로 했다. 그 대신 백엔드가 딱 두명이라 머지는 리뷰를 한 다른 쪽 한명이 해주는 걸로 하였다.  
커밋 메시지는 이전에 우테코에서 했던(지금은 모든 커밋 메시지를 이렇게 쓰고 있다.) `AngularJS Git convention`을 따라서 하였다.  

### FE 
같은 대상을 지칭하는 용어가 FE와 다르다는 것을 느꼈다. 
프로젝트로 FE 코드를 처음 봤는데, 리액트를 조금 할 줄 안다고 생각했는데도 어떻게 작동되는지 잘 모르겠다.  
그냥 팀원분들을 믿는 수밖에..! 

## 이미지 저장
### multipart form
이미지를 DB에 저장하는 방법은 multipart form data를 이용하거나 Base 64 encoding을 하는 방식이 있는 것 같았다.  
사실 이미지 저장은 직접적으로 구현해 본 적은 없었고, 대충 방법만 알고 있었다. 
이전 백엔드 개발하신 분들은 Base 64로 인코딩을 이용하여 스트링 타입으로 저장하셨던 것 같다.
하지만, 아무리 스트링값이라고 해도 오버헤드가 있고 이미지 수정을 하는 경우도 있을 것 같아서 따로 스토리지를 구축하여 해당 스토리지 URL만 저장하는 방식으로 
이미지 저장 및 수정을 구현하였다. 

DB를 구축할 때 AWS RDS를 이미 구축한 상태였으므로, S3를 구축하였고, `org.springframework.cloud`의 `spring-cloud-starter-aws`를 사용하여 
공식 s3 API를 사용하였다. 

대충 소스코드는 이렇게 된다. 
```java
@Service
@RequiredArgsConstructor
@PropertySource("classpath:application-aws.yml")
public class S3Uploader {
	private final static List<String> IMAGE_EXTENSIONS = Arrays
		.asList(".jpg", ".jpeg", ".gif", ".png", ".img", ".tiff", ".heif");
	private final static String TEMP_FILE_PATH = //////// 비공개 /////////
	private final AmazonS3Client amazonS3Client;
	@Value("${cloud.aws.s3.bucket}")
	public String bucket;

	public File convert(MultipartFile file) {
		File convertFile = new File(TEMP_FILE_PATH + file.getOriginalFilename());
		try {
			if (convertFile.createNewFile()) {
				try (FileOutputStream fos = new FileOutputStream(convertFile)) {
					fos.write(file.getBytes());
				}
				return convertFile;
			}
		} catch (Exception e) {
			System.err.println(e.getMessage());
		}
		return null;
	}

	public String upload(File uploadFile, String dirName, String id) {
		String fileName = dirName + "/" + id;
		String uploadImageUrl = putS3(uploadFile, fileName);
		deleteLocalFile(uploadFile);
		return uploadImageUrl;
	}

	public boolean isValidExtension(File uploadFile) {
		String fileName = uploadFile.getName();
		String extension = "." + fileName.substring(fileName.toLowerCase().lastIndexOf(".") + 1);
		if (!IMAGE_EXTENSIONS.contains(extension)) {
			deleteLocalFile(uploadFile);
			return false;
		}
		return true;
	}

	public void delete(String dirName, String id) {
		deleteFromS3(dirName + "/" + id);
	}

	private void deleteFromS3(String key) {
		amazonS3Client.deleteObject(bucket, key);
	}

	private String putS3(File uploadFile, String fileName) {
		amazonS3Client.putObject(
			new PutObjectRequest(bucket, fileName, uploadFile).withCannedAcl(
				CannedAccessControlList.PublicRead));
		return amazonS3Client.getUrl(bucket, fileName).toString();
	}

	private void deleteLocalFile(File uploadFile) {
		if (!uploadFile.delete()) {
			System.err.println(ErrorCase.FAIL_FILE_DELETE_ERROR);
		}
	}
}
```
`s3`의 `bucket`은 클래스 변수이지만, 배포 환경에 따라 바뀔 수 있으므로 프로퍼티를 분리하였다. 
하지만, 보안 상 이슈로 깃헙에 직접적으로 올릴 수 없으니 메인 프로퍼티와는 분리하였다.  
또한, [`AWS` 자격 증명](https://docs.aws.amazon.com/ko_kr/sdk-for-java/v1/developer-guide/credentials.html)도 해야 하므로
프로퍼티를 이용하는 게 좋겠다고 생각했다. (`AWS SDK for JavaSystemPropertiesCredentialsProvider`, [여기](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html?com/amazonaws/auth/SystemPropertiesCredentialsProvider.html) 참고)  
```properties
cloud:
  aws:
    s3:
      bucket: //// 비공개 ////
    region:
      static: ap-northeast-2
    stack:
      auto: false
    credentials:
      accessKey: //// 비공개 ////
      secretKey: //// 비공개 ////
      instanceProfile: true
```


`AWS`에서 권장하는 [로컬 파일 형태로 키들을 저장하거나 환경 변수를 이용한 자격 증명](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html)은, 당시 배포 환경이 정해지지 않아서 고려하지 않았었다. 

파일을 직접 `s3`로 `put`하기 이전에 파일 형태로 바꿔서(`S3`에 `Multipartfile` 타입은 전송이 안된다.) `classpath` 어딘가에 미리 저장해 놓는다. `null`을 리턴하는게 마음에 안 들긴 하지만,
컨트롤러 쪽에서 `null`을 받으면 에러를 출력하게 해놓았다. 
또한 업로드 이전에 최소한의 보안을 위해 파일 확장자 검사를 `white-list` 형식으로 해주었다.  

찾아보니 공식 지원 `JAVA API`에 멀티파티 업로더가 있던데, 나중에는 그걸 써봐야 겠다.  
전체적인 흐름은 [이 블로그](https://jojoldu.tistory.com/300)를 많이 참고하였다.  

### S3

### S3 버전 관리
`S3`로 새로운 이미지 등록, 기존 이미지 삭제는 잘 구현이 되는데, 수정이 잘 되지 않아서 애를 먹었다.  
찾아보니, `S3`는 버전이라는 게 존재하고, 이미지를 수정하면 맨 처음 생성한 이미지의 `URL`과는 달라(`URL` 뒤에 추가 파라미터가 붙는 형태)진다.

따라서 이미지를 수정한 후에 `RDS`에도 바뀐 `URL`을 넣어주게 하였고, 버전이 바뀐 이미지나 삭제된 이미지를 주기적으로 영구 삭제하는 수명 주기 규칙을 만들었다. 
해당 내용은 [여기](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)를 참고하였다.  

## JPA
### RDS 연결
사실 `RDS` 연결은 그냥 `MySQL Driver` 연결과 똑같다.  
`url`만 `RDS URL`로 써주면 되는데, [사이트](https://mvnrepository.com/artifact/mysql/mysql-connector-java)에서 버전에 맞는 디펜던시를 추가해주면 된다. 

### 매핑 
`JPA`에서 매핑의 종류가 꽤 많아서 자주 헷갈렸다. 결론적으로 `@OneToMany`와 `@ManyToOne`를 사용하였는데, 기준은 해당 엔티티가 앞이다. 

#### 1. @OneToMany
어노테이션이 붙는 엔티티(편의상 '나'라고 하겠다.)가 One이고, **나를 기준으로 여러개**가 연결되는 것이다.  
따라서 `List`나 `Set` 등의 `Collection` 인터페이스에 어노테이션이 붙어야 한다. 

사실 전통적인 `RDBMS`에서는 엔티티 매핑을 양방향으로 구성하지 않는다. 이는 리스트 형태이기 때문에 당연한 말이다. 
따라서 `@OneToMany`는 `ORM`의 장점이다. 원래라면 `@ManyToOne`에서 설정된 것처럼 구성하고, 참고하는 테이블에서 `FK`를 모두 찾아서 리스트 등으로 가져와야 한다. 

```java
@OneToMany(mappedBy = "portfolio", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
```

이런 식으로 어노테이션이 붙는데, 이렇게 되면 `1:M`을 `양방향`으로 접근하는 것이다. 만약 `dto`로 값을 받지 않고 `entity`로 그대로 받는다면, 상황에 따라 리턴값이 재귀적으로 호출될 수 있다... 
여러 해결 방법이 있었지만 그냥 `dto`를 만드는 게 좋다고 생각하여 만들어주었다.

내가 `@OneToMany` 이므로, `@ManyToOne`에서 나를 해당 엔티티 객체로 접근하게 된다.  
따라서 저쪽에서는 변수 **타입**이 나일 것이고, `mappedBy`에는 변수**명**을 적어주면 된다.
따라서 에러가 난다면, `@ManyToOne` 어노테이션이 붙어있는 엔티티 코드를 확인해 봐야 한다.  

`fetch`는 `LAZY`와 `EAGER`을 쓰는데, `EAGER`은 한 객체가 바뀌면 연관된 객체가 거의 준 실시간으로 바뀌도록 쿼리가 나가게 된다. 
이는 성능에 영향을 미치므로 거의 `LAZY`를 쓰는데, 모든 호출에서 객체를 호출하면 그때그때 마다 새로운 업데이트 값이 생기길 원할 수 있으니 그 때는 `EAGER`을 쓰는 게 좋다.

#### 2. @ManyToOne
이번엔 내가 Mnay고, 상대방이 One이다. 따라서 변수 타입은 위에서 말한 것처럼 엔티티 클래스이다. 
```java
@ManyToOne
@JoinColumn(name = "users_id")
```
위에서 말한 것처럼 전통적인 `RDBMS`에서는, 참조**되는** `PK`를 참조**하는** 쪽에서 `FK` 형태로 포함한다. 
따라서 `@JoinColumn`에는 실제 데이터베이스의 테이블에서 `FK`로 설정되어있는 **컬럼명**을 써주어야 한다. 
실제로 에러가 나면 `@OneToMany`와는 달리 테이블에 키가 없다고 뜨며, 이 때는 코드를 살펴보기보다는 `MySQL`의 테이블 설정을 살펴보는 게 더 도움이 되었다. 

#### 3. @ManyToMany
사실 다대다 매핑에 `@ManyToMany`를 사용할 수 있으나, 실제로 적용해보면 필요 이상으로 쿼리가 나가는 경우가 많아 성능 이슈가 있다. 
따라서 전통적인 다대다 매핑에서 처럼 연결 테이블을 엔티티로 승격하는 방식을 사용하여 위의 `@OneToMany`와 `@ManyToOne`로 연결하였다. 

### Specification
사실 간단한 `CRUD API`만을 위한 레포지토리가 필요한 경우에는 `extends CrudRepository<엔티티클래스, PK타입>`으로 상속받으면 된다.  
거기에 추가로 페이지네이션이나 정렬을 사용할 경우에는 `JpaRepository`를 많이 상속한다. 

하지만, 이번 프로젝트에서 페이지네이션 검색 조건이 선택적이었고, 많았다. 검색 조건 은 0개부터 5개까지 였고, 값이 들어오지 않으면 검색 조건으로 넣지 않았어야 했다.  
따라서 아무것도 선택 안했을 때 ~ 모두 선택 했을 때의 검색 조건 조합의 경우는 32가지였다. 32개의 `if`문을 만들 수는 없었고, 하나는 내부에 `OR` 조건이 필요했다. 
`findBy` 뒤에 `And`와 `Or`을 넣을 경우 우선순위나 or 범위를 설정할 수 없다. 

그래서 찾아보니 `JPA`에서 검색 조건을 추상화한 객체인 `Specification`을 사용하면 될 것 같았다. 

```java
public interface ProjectRepository extends JpaRepository<Project, Long>, JpaSpecificationExecutor<Project> {
	Page<Project> findAll(Specification<Project> specification, Pageable pageable);
}
```
레포지토리를 상속 받을 때 `JpaSepcificationExecutor<엔티티클래스, PK타입>`을 하나 더 상속하면 파라미터로 `Specification`을 넣을 수 있다.

레포지토리를 이렇게 정의하고, 
```java
public interface ProjectRepository extends JpaRepository<Project, Long>, JpaSpecificationExecutor<Project> {
	Page<Project> findAll(Specification<Project> specification, Pageable pageable);
}
```

`Specification` 클래스를 따로 만들어 준다. 

```java
import org.springframework.data.jpa.domain.Specification;

public class ProjectSpecification {
	public static Specification<Project> ByCategory(final String category) {
		return (Specification<Project>) ((root, query, builder) ->
			builder.equal(root.get("category"), category));
	}

	public static Specification<Project> ByRecruitmentField(final String recruitmentField) {
		return (Specification<Project>) ((root, query, builder) ->
			builder.equal(root.get("recruitmentField"), recruitmentField));
	}

	public static Specification<Project> ByRegion(final String region) {
		return (Specification<Project>) ((root, query, builder) ->
			builder.equal(root.get("region"), region));
	}

	public static Specification<Project> ByProjectSection(final String projectSection) {
		return (Specification<Project>) ((root, query, builder) ->
			builder.equal(root.get("projectSection"), projectSection));
	}

	public static Specification<Project> ByKeyword(final String keyword) {
		Specification<Project> specification = ByKeywordInContent(keyword);
		specification = specification.or(ByKeywordInTitle(keyword));
		return specification;
	}

	private static Specification<Project> ByKeywordInContent(final String keyword) {
		return (Specification<Project>) ((root, query, builder) -> builder
			.like(root.get("content"), "%" + keyword + "%"));
	}

	private static Specification<Project> ByKeywordInTitle(final String keyword) {
		return (Specification<Project>) ((root, query, builder) ->
			builder.like(root.get("title"), "%" + keyword + "%"));
	}
}
```

이후에 서비스에서 적당히 구현해주면 된다. 

```java
public ProjectListResponseDto getProjectsList(Pageable pageable, String category,
    String recruitmentField, String region, String projectSection, String keyword) {
    Specification<Project> specification = Specification.where(null);
    if (category != null && !category.isEmpty()) {
        specification = specification
            .and(Specification.where(ProjectSpecification.byCategory(category)));
    }
    if (recruitmentField != null && !recruitmentField.isEmpty()) {
        specification = specification.and(
            Specification.where(ProjectSpecification.byRecruitmentField(recruitmentField)));
    }
    if (region != null && !region.isEmpty()) {
        specification = specification
            .and(Specification.where(ProjectSpecification.byRegion(region)));
    }
    if (projectSection != null && !projectSection.isEmpty()) {
        specification = specification
            .and(Specification.where(ProjectSpecification.byProjectSection(projectSection)));
    }
    if (keyword != null && !keyword.isEmpty()) {
        specification = specification
            .and(Specification.where(ProjectSpecification.byKeyword(keyword)));
    }
    return pageToListResponseDto(projectRepository.findAll(specification, pageable));
}
```
 

### Criteria
검색 조건 페이지네이션 부분까지는 동적 쿼리를 사용할 일이 없었다. 
하지만, 막판에 `SQL`문에서의 `group by`나 가상 필드가 필요했다. 따라서 동적 쿼리를 사용하기로 했다.  

`JPA 2.0`부터 `JPQL`을 자바 코드로 작성할 수 있는 빌더 클래스를 제공한다. 
즉, 동적 쿼리를 사용하기 위한 JPA 라이브러린데, 이게 바로 `CriteriaBuilder`이다. 
이를 이용하면 좀 더 넓은 범위에서의 다양한 쿼리(`group by`, `SUM`, `AVG`, `ORDER BY` 등..)를 사용할 수 있다. 

```sql
SELECT tag_id, SUM(score) from tag_history GROUP BY tag_id ORDER BY SUM(score) LIMIT 10;
```
이러한 `SQL` `select`문이 있을 때, 

```java
CriteriaBuilder builder = entityManager.getCriteriaBuilder();

CriteriaQuery<TagHistoryGroupByDto> query = builder.createQuery(TagHistoryGroupByDto.class);

Root<TagHistory> root = query.from(TagHistory.class);query.groupBy(root.get("tag"));
query.multiselect(root.get("tag"), builder.sum(root.get("score")));
query.orderBy(builder.desc(root.get("score")));

return entityManager.createQuery(query).getResultStream().limit(10);
```
이런 식으로 처리할 수 있다. 

`TagHistoryGRoupByDto`는 `tag`와 `SUM(score)`로 이루어진 `dto`로 보면 된다.  
원래는 `createQuery(query).setMaxResult(10).getResultStream()`을 하려고 했는데, 그러면 `ORDER BY`가 제대로 작동하지 않는다. 
`ORDER BY`이전에 `limit`이 들어가는 것 같다. 
그래서 그냥 `stream`으로 받아서 `limit`을 해주었다. 


## 스케줄링
스프링에서 스케줄링은 여러가지 방법이 있다. `Spring Scheduler`나 `Quatz`를 이용할 수도 있다.   
나는 그냥 `Spring Scheduler`를 사용하기로 했다. 이전에 써보기도 했고, `spring boot starter`에 기본적으로 제공되서 어노테이션만 붙여주면 되기 때문이다. 

```java
@SpringBootApplication
@EnableScheduling
public class PickmeupApplication {

	public static void main(String[] args) {
		SpringApplication.run(PickmeupApplication.class, args);
	}

}
```
메인 메서드에서 `@EnableScheduling` 어노테이션을 붙여준다. 
```java

@AllArgsConstructor
@Service
public class TagService {
	private final TagRepository tagRepository;
	private final TagHistoryRepository tagHistoryRepository;

	@Scheduled(cron = "0 0 0/3 * * ?") // save score history every 3 hours
	public void updateScore() {
		Timestamp currentTime = new Timestamp(System.currentTimeMillis());
		System.out.println("[*] " + currentTime + ": update score START");
		tagRepository.findAll().forEach(tag -> addTagHistory(tag, currentTime));
		System.out.println("[*] " + currentTime + ": update score END");
	}

	@Scheduled(cron = "0 0 6 * * ?") // everyday at 6AM
	public void removeOldHistory() {
		Timestamp currentTime = new Timestamp(System.currentTimeMillis());
		System.out.println("[*] remove history START");
		ZonedDateTime zonedDateTime = currentTime.toInstant().atZone(ZoneId.of("UTC"));
		Timestamp standardTime = Timestamp
			.from(zonedDateTime.plus(-1, ChronoUnit.DAYS).toInstant()); // before 1 day
		tagHistoryRepository.deleteOldScore(standardTime);
		System.out.println("[*] remove history END");
	}

...

}
```

`Quatz`는 `IoC`측면에서 주입성이 좀 떨어진다는 의견이 많길래 간단한 스케줄링이라 `Spring Scheduler`만 적용하였다. 
사실 DB의 양이 많아지면 작업이 느려질 수도 있고, 트래픽 집계 같은 경우는 `Spring Batch`를 더 많이 사용하는 편이다.  
일단은 트래픽이 그렇게 많은 편이 아니라 직접 `DB`에 접근하여 트래픽 집계 및 오래된 트래픽을 버리는 식으로 적용했는데,
나중에 `CI`적용한다면 `Spring Batch`로 같이 마이그레이션 하려고 한다. 


## 배포
`AWS` 배포는 직접적으로 서버에 접근할 수 있으므로 리눅스가 익숙하다면 매우 쉽다.
 
### EC2
`EC2`에서 인스턴스를 프리티어로 생성한다. `AMI`는 `Amazon Linux 2 AMI (HVM), SSD Volume Type -`으로 설정하였고, 
인스턴스에 직접적으로 들어가는 방법은 키 페어(`.pem`)와 `ssh`를 이용하면 된다. 

인스턴스가 들어가보면 `yum` 및 `rps`는 되지만, `apt-get`같은 우분투 기반 명령어는 먹히지 않는다.

#### 1. JAVA 설치  
```bash
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```
`devel`을 제외하면, `java`가 `jre`를 가리키게 되어, `gradlew build` 실행 시에 에러가 난다.

#### 2. git clone

#### 3. property .yaml 파일들 수정
아까 `aws` 프로퍼티나 그 외 깃헙에 보안 이슈 때문에 올리지 못한 프로퍼티들을 직접 수정해준다. 

#### 4. gradlew 권한 설정
`git clone` 으로 가져온 `gradlew`는 실행 권한이 없으므로 설정해준다.
 
#### 5. ./gradlew build
빌드가 성공하면 `./build/libs`에 `.jar` 파일이 생긴다. 

#### 6. jar 파일 실행
```bash
nohup java -jar ./build/libs/파일이름.jar &
```

데몬 형태로 실행시키기 위해 `nohup`으로 실행시켰고, 백그라운드(`&`)로 실행시켰다. 
 

### 자동 스크립트
위의 1~6과정을 매번 배포 때마다 하기는 귀찮으므로, 자동 스크립트를 써준다. 

```bash
#!/bin/bash

PICKMEUP_PROJECT=/home/ec2-user/...비공개.../
cd $PICKMEUP_PROJECT/pick-me-up-server
echo "[*] git update..."
git fetch --all
git reset --hard origin/main
git pull --force
echo "[*] copying properties..."
cp -f ../application.yml ./src/main/resources/application.yml
cp -f ../application-aws.yml ./src/main/resources/application-aws.yml
echo "[*] project building..."
chmod 755 gradlew
./gradlew clean build
echo "[*] PID checking..."
CURRENT_PID=$(pgrep -f pickmeup)
echo "current PID : $CURRENT_PID"

if [ -z $CURRENT_PID ]; then
	echo "[*] no running appplication"
else
	echo "[*] kill -15(SIGTERM) $CURRENT_PID"
	kill -15 $CURRENT_PID
	echo "[*] Please wait..."
	sleep 30
fi

echo "[*] new application deploying!"
JAR_NAME=$(ls ./build/libs/*.jar)
echo "[*] JAR name : $JAR_NAME"
cd ../
rm nohup.out
nohup java -jar ./pick-me-up-server/build/libs/*.jar &
echo "[*] new application Starting..."
sleep 30
tail nohup.out
```

`application.yml`과 `application-aws.yml`은 pull 후에 덮어쓰므로, 다음 배포 시 `pull` 할 때에 `push`하지 않아서 에러가 난다.  
따라서 `local`을 `overwrite`하는 경우를 찾아보니 [관련 스택오버플로우](https://stackoverflow.com/questions/1125968/how-do-i-force-git-pull-to-overwrite-local-files) 글을 찾았다.  
리모트의 가져오기만 하는 `fetch` 사용한 후에, `reset --hard`로 해당 컬로 돌아가고, `pull --force`를 한다.  
사실 `pull --force`는 필요없긴 한데, 이후 안정성을 위해 넣어준다. 

그리고 이전 `nohup.out` 파일이 계속 쌓이다보니 지워주었는데, 나중에 로깅 스택을 구축하면 다른 곳에 쌓일 거라 생각해 이렇게 하였다.  

가끔 이전 `.jar`파일을 실행한 프로세스가 아직 완전히 종료되지 않았는데, 다시 배포하여 새롭게 생성된 `.jar` 파일을 실행하는 경우,
포트가 겹치는 문제가 발생하여 30초 동안 `sleep`하였으나, 메모리가 부족한 경우가 발생하였다. 

여러 해결방법을 시도한 끝에 스와핑 방법으로 해결하였다. 
구체적인 스와핑 방식은 [AWS 글](https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/)을 참고하였다. 


### FE 연동
FE 연동에 있어서도 약간의 이슈들이 있었다. 
#### CORS
`nginx`로 한꺼번에 배포하지 않고 이미 `FE`가 `vercel`으로 떠있었기 때문에 도메인이 달라 `CORS` 에러가 발생하였다.
이를 해결하기 위해서는 여러 방법이 있는데, [baeldung](https://www.baeldung.com/spring-cors)에 잘 정리된 포스트가 있어서 참고하였다. 
나는 `@Configuration` 빈의 `WebConfigurer`을 구현하였다. 

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**")
			.allowedOrigins("https://pickmeup.site", "https://pick-me-up.vercel.app")
			.allowedMethods("GET", "POST", "DELETE", "PUT", "PATCH")
			.exposedHeaders("Location");
	}
}
```
 
처음에 설정했는데도 `DELETE` 메서드만 `CORS` 에러가 나서 살펴보니 
`allowedMethods`를 써주지 않으면 기본적인 `http method`들만을 허용한다고 공식 문서에서 본 것 같다.

이미지를 등록하거나 수정할 시에 `Location` 헤더로 `S3`의 `URL`을 보내므로 헤더 허용 설정을 추가해주었다. 
 
#### https

`CORS` 설정을 해도 배포 후 에러가 떴다. `vercel` 앱은 `https`로 설정되어 있는데, 백은 `http`로 배포했으므로 보안 상 생기는 문제였다. 

#### 1. 도메인 구매
[도메인 구입 사이트](https://www.freenom.com/en/index.html?lang=en)에서 `0$`로 골라서 구입했다. 
12개월(1년)을 무료로 이용할 수 있다고 한다.  

#### 2. Route53
1에서 구입한 도메인을 `AWS Route53`에서 호스팅 영역으로 추가한다. 
추가 후 1에서 구입한 도메인의 `NS`를 `AWS`에서 보이는 `NS`로 교체해야 한다. 
1번 사이트에서 `Service > My Domains > Manage Domain > Management Tools > Nameservers`로 들어가
`Route 53`에서 보이는 `NS`로 수정해주면 된다.

#### 3. 인증서 발급
인증서는 `Let's Encrypt`나 `ACM`을 이용할 수 있는데, `Let's Encrypt`는 매년 갱신이 필요하고,
`ACM`이 `Route53` 설정으로 쉽게 할 수도 있어서(`CNAME` 레코드 설정만 하면 된다.) `ACM`을 이용하였다. 

#### 4. 로드밸런서 구성
3에서 만든 인증서를 통해서 `EC2` 로드밸런서를 구성하여야 한다.  
**리스너**는 로드밸런서가 **외부에서** 받아들이는 포트인데, 이를 3에서 만든 `SSL` 인증서를 통해 만들어 준다. 
그리고 보안 규칙을 통해 `http 80`으로 들어온 트래픽을 자동으로 `https 443`로 리다이렉팅 해준다. 

참고로 비슷하게 `EC2` 보안 규칙이 설정 방식이 비슷해서 헷갈렸었다. 
보안 규칙 중 인바운드 규칙은 안에서 어떤 포트로 통신하는지를 고려해야 한다.  
나같은 경우는 스프링부트가 기본 포트 `8080`으로 설정되어 있으므로 `TCP 8080`을 설정해주어야 한다.
나머지 `http 80`이나 `https 443`같은 경우도 기본으로 설정해준다.  
또한 `ssh`로 인스턴스 쉘에 접근을 하므로(배포 때처럼) 이미 `ssh 22`가 설정되어 있다.  
{: .notice}


* * *

# 느낀점
개인적으로 아쉬운 점들이 몇 개 있다. 시간 상 테스트 코드 짜기, 로깅이나 모니터링, `CI/CD` 등을 하지 못했는데, 테스트 코드는 꼭 했어야 한다고 생각한다.  
  
그리고 만약, 쿠버네티스를 통해 배포를 했으면, 스케줄링은 자바 `SehcduledExecutorService`를 사용하기 보다는
쿠버네티스의 크론잡을 이용할 걸 그랬다는 생각이 든다. 아무 생각 없이 `aws`로 해야겠다고 생각하고 `ec2`로 배포해버려서,
깃 액션이고 `CI/CD`고 뭐고 이용하지 못했다. 배포도 코드 짤 때처럼 계획적으로 해야했는데 말이다..  

아직 `FE`팀도 미완성이라 완성 후에 베타 테스트를 거칠 예정이니, 여름방학 때 부족한 점들을 개선해 봐야겠다.  
