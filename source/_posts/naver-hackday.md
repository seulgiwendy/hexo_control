---
title: 2018 네이버 CAMPUS HACKDAY 참가기
date: 2018-05-19 09:21:10
categories:
    - naver
    - hackathon
---

### 2018 네이버 캠퍼스 핵데이 참가 후기 

네이버가 주최하는 대학생을 위한 해커톤 행사인 [2018 네이버 캠퍼스 핵데이](https://github.com/NAVER-CAMPUS-HACKDAY/common)에 다녀왔다. 매년 여름 겨울 두 차례 실시되는 이 행사는 소프트웨어 개발에 관심이 있고 기초 실력(?) 을 가진 대학생이라면 누구나 참가할 수 있는 행사로 이번이 다섯번째다. 우수 참가자에게는 **기술면접 후 네이버 인턴 채용 기회** 가 부여된다.

출발전 간단한 오리엔테이션이 이뤄졌는데 *매 행사마다 새로운 최고 경쟁률 기록을 갈아치우고 있다* 고 한다. 뭐.... 나중에 느낀 거지만 이정도의 **고품격 해커톤을 운영**한다면 당연한 결과라고 생각된다. 

---

#### 오리엔테이션과 출발 전 간단 미팅 

![오리엔테이션](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/KakaoTalk_Photo_2018-05-19-09-30-10.jpeg)

오리엔테이션은 이제 [우리집 안방처럼](http://nter.naver.com/naverletter/textyle/135326) [편안하고](http://startupall.kr/blog/2017/09/22/2017-%EC%A4%91%EA%B5%AD%EC%9D%98-%ED%95%9C%EA%B5%AD%EC%9D%B8%EC%9D%84-%EA%B0%9C%EC%B5%9C%ED%95%A9%EB%8B%88%EB%8B%A4/) [아늑한](http://startupall.kr/blog/2018/03/09/%EC%8B%A4%EB%A6%AC%EC%BD%98%EB%B0%B8%EB%A6%AC%EC%9D%98-%ED%95%9C%EA%B5%AD%EC%9D%B8-2018/) 네이버 그린팩토리 커넥트홀에서 진행됐다. 

단체로 이동할 때 통상적인 주의사항과 **술 마시면 퇴소** 라는 규정을 알려줬다. 가보면 알겠지만 술 사갖고 올 만한 슈퍼도 없긴 하다. 오리엔테이션 내용이 별로 없었던 이유는 이 행사 자체가 팀 별로 멘토의 주도 하에 고도의 자유도를 부여하는 행사로서 이미 멘토들의 판단에 따른 주의사항이 각종 연락 채널(Slack, 익명 카톡방(...) 등등...) 으로 전파가 되어 있는 상황이었다. 

다른 팀의 이야기를 들어보니 모이기 전 **이미 몇 번의 사전 준비 모임**과 멘토와의 식사, 멘티들끼리의 식사 등을 가진 팀이 있었고, 프로젝트에 대해서는 (기본 요구사항 외에) 한 줄의 안내도 받지 못한 팀도 몇 군데 있었다. 팀의 성향 그리고 멘토의 성향에 따라 차이를 보인 부분으로, 사실 같은 핵데이 행사에 참여했다고 하더라도 본인이 참가한 소주제에 따라 경험이 많이 달라졌을 것 같다. 

우리 팀은 사전 준비모임을 갖지 않았고(그 이유도 후술한다) 출발 직전 멘토님과 점심 식사를 가졌다. 멘토님은 네이버 쇼핑의 데이터정제파트에 근무하고 계시고 굳이 백 / 프론트를 나누자면 백엔드 개발자에 더 가까운 분이었다. 

식사 자리는 극도의 긴장감 속에(...) 식은땀이 줄줄 흘렀다. 사실 식사 뿐만 아니라 멘토님 앞에서 기술적인 얘기 하려고 하면 식은땀이 막 났다. *되도 않는 헛소리 하고 있는건 아닐까 하고...*

식사 자리에서의 멘토님의 말씀 중 인상적인 부분을, 공유할 수 있는 범위 내에서 공유해 보겠다. 

- 회사에서 **채용을 위한 행사** 로 비치지 않기를 원한다. 너무 과열되지 않기를 바라는 것 같다. 
- 지나치게 구현 완성도에 집중하지 않았으면 좋겠고 편안한 분위기 속에서 즐코딩(멘토님이 즐겨쓰시는 표현)하셨으면 좋겠다. **과정이 중요하다.** 
- 우수 참가자에게는 면접 후 인턴 채용 기회가 부여되는데 **면접이 어렵다.**
- (참가자 선발 기준은?) **GitHub 보고 뽑았다.** 전공 학점 등등 전혀 안 봤다. 
    - GitHub를 볼 때 두 가지를 봤다. **커밋 빈도** 와 **커밋 메시지** 를 봤다. 커밋을 얼마나 자주 많이 했는지를 봤고 메시지를 극도로 잘 쓰진 않아도 의미있게 작성하는지를 봤다. 1번 커밋, 2번 커밋, ㅋㅋㅋ 이런 식으로 남긴 것은 좋게 평가하지 않았다. 
- 팀별로 약간명을 후보자로 제출하면 인사팀에서 최종 스크리닝하는 절차를 거쳐 **세 명을 추렸다.**

*(팀원 선발에 관한 부분은 멘토님과 팀의 개인 성향으로 획일적인 게 아니다. 실제로 최근 한달간 커밋 몇 건 없이 핵데이 오신 분들도 있다)*

특히 내 경우 이번 여름 졸업이 아닌 이번 가을학기를 이수해야 하는 입장이라 참가 전에도 *헛수고하는 건 아닌가* 하고 걱정을 좀 했는데 나같은 경우 1월 입사가 가능하게끔 입사 시기도 배려해 준다고 하셨다. 

이런 김칫국 끓이는 질문을 먼저 드릴 수 없었는데 *다들 몇학기 남았어요? 이번에 졸업이에요?* 하고 먼저 따스히 물어봐주셔서 답답함을 해소하고 새우튀김에 집중할 수 있었다. 

---

#### 입소, 코딩, 식사 , 코딩, 야식, 코딩, 식사 

![중앙고속도로](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/KakaoTalk_Photo_2018-05-19-10-14-42.jpeg)

분당에서 춘천 가는 길이 참 멀게 느껴졌다. 마석 쯤 왔겠거니 하고 창밖을 보니 미사리라서 소름이 좀 돋았다. 

어릴때부터 수도없이 달린 길인데 엄청 새롭게 느껴졌다. 

![커넥트원](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/40226631-246fe304-5ac7-11e8-89f0-1742d9b4af6c.JPG)

네이버 커넥트원 내부 시설은 **사진 촬영을 엄금** 했다. 외부 풍경사진 정도, 건물 내부시설이 전혀 비치지 않는 얼굴 사진과 프로젝트 진행 사진 정도만 허용이 됐고 이외에는 전혀 사진 촬영을 할 수 없었다. 남의 건물에 왔으면 하지 말라는 것은 무조건 하지 않는 게 좋으므로 내부시설 사진은 찍지 않았다. 

**이후 일정**

숙소에 짐을 풀고 작업공간으로 내려왔다. **코딩파티가 시작되었다.** 

이후로는 일정이랄 것도 없고 계속 코딩과 식사를 반복했다. **밥을 엄청 잘 준다.** 네이버가 이번 행사를 기획한 진짜 목적은 굶주린 청년들의 배를 채워주려는 것이 아니었을까 싶을 정도로 먹을 것 제공이 잘 됐다. 

식사는 시설이 아니니까 사진 촬영을 좀 했다. 

![1일차저녁](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/KakaoTalk_Photo_2018-05-19-10-42-02.jpeg)

첫째날 저녁인데 **무려 셰프님이 철판을 들고 나오셔서 닭갈비를 볶아주신다.** 첫 식사부터 엄청난 퀄리티에 압도되었는데 이건 시작에 불과했다. 

![2일차아침](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/KakaoTalk_Photo_2018-05-19-10-42-11.jpeg)

**아니 내가 어디 호텔에 와 있나?** 2일차 아침이다. 

![마지막](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/KakaoTalk_Photo_2018-05-19-10-42-20.jpeg)

마지막까지 이런 식사를.... 너무 감덩......


이외에도 밤에 **즉석에서 튀긴 순살 치킨** 이 제공됐다. 너무 행복했다. 


**프로젝트 진행**

프로젝트 진행은 팀 별로 배정된 공간에서 각자의 개발 장비를 펼쳐놓고 진행한다. 팀 별로 공간이 배정되어있지만 이번 행사를 위해 **커넥트원 시설을 통째로 사용** 했으므로 커넥트원 내부에서 팀별로 코딩을 할 수 있는 장소라면 어디든지 사용할 수 있었다. 

우리 팀도 어디로 옮겨볼까? 했지만 주행사장이 너무 안락하고 좋아서 그대로 앉아서 하기로 했다. 시설에는 프로젝트에 몰입할 수 있게 설계한 공간들이 많았고, 사진을 촬영하고 싶었지만 사진 촬영이 엄금되어 있어 사진을 촬영하지 못했던 점 이해해 주시길 바란다. 

우리팀의 주제는 **단위 테스트에 기반한 실시간 공지사항 플랫폼** 이었다. 단위 테스트의 중요성이 크게 강조됐다. TDD까지는 아니어도 주요 컴포넌트에 대한 단위 테스트가 이뤄져야함을 멘토님이 늘 강조하셨다. 

프로젝트의 요구사항을 그대로 옮기기는 어려울 것 같고, 내가 작성한 코드를 중심으로 간단한 해설과 우리 주제에 대한 내 생각을 곁들여본다. 

**프로젝트의 결과물은 1인 1제출을 원칙으로 했다(우리팀의 경우).**

[배포된 프로젝트 사이트 구경가기](http://naveradm.wheejuni.com)

SupportedFileTypes.java

```java
@Getter
public enum SupportedFileTypes {

    XLS("xls", "application/vnd.ms-excel"),
    XLSX("xlsx", "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"),
    CSV("csv", "text/csv"),
    TSV("tsv", "text/tsv"),
    TXT("txt", "text/plain"),
    PNG("png", "image/png");

    private String extension;
    private String mimeType;

    SupportedFileTypes(String extension, String mimeType) {
        this.extension = extension;
        this.mimeType = mimeType;
    }

    private boolean isMatchingType(String extension) {
        return extension.equalsIgnoreCase(this.extension);
    }

    public static boolean isSupportedFile(MultipartFile file) {
        String filename = file.getOriginalFilename();
        return Arrays.stream(SupportedFileTypes.values()).anyMatch(t -> t.isMatchingType(FilenameUtils.getExtension(filename)));
    }
}
```

프로젝트의 요구사항 중, 특정 파일 타입에 대해서만 업로드를 허용하는 사항이 있었다. 

물론 프론트에서도 유효성 검사를 시행하지만 이번 주제의 경우 프론트보다는 서버단의 구현이 훨씬 중시되는 것이 사실이기 때문에 백엔드에서의 검증 로직을 구현해봤다. 

FileService 에서 파일 확장자에 대한 검증을 수행하는 것도 좋다고 생각했지만, 파일 확장자와 MIME type 등을 하드 코딩해서 구현한다면 차후 허용되는 확장자가 변경되는 상황에 대해 유연하게 대응하기 어렵다고 판단, 파일 형식을 Enum으로 구현하기로 결정했다. 

파일 형식에 대한 Enum 구현을 마치고 나니 Multipartfile Request 자체에 대한 검증 로직을 파일타입 객체에서 전담하게 하는 것이 설계상 맞는 판단이라고 생각되어 isSupportedFile(MultipartFile file) 을 통한 파일 확장자 유효성 검증까지 맡겼다. 

파일 타입에 대한 검증은 Java 8의 Stream API를 적극적으로 사용했다. 

SupportedFileTypesTest.java

```java

public class SupportedFileTypesTest {

    private MultipartFile file;
    private MultipartFile fileExcel;

    @Before
    public void setUp() {
        this.file = mock(MultipartFile.class);
        when(this.file.getOriginalFilename()).thenReturn("testfile_withUnderBars_whynotworking.csv");

        this.fileExcel = mock(MultipartFile.class);
        when(this.fileExcel.getOriginalFilename()).thenReturn("testfile_multiperiods.xlsx");
    }

    @Test
    public void supportedFileTypes_fileExtensionCheck() {
        assertTrue(SupportedFileTypes.isSupportedFile(this.file));
        assertTrue(SupportedFileTypes.isSupportedFile(this.fileExcel));
    }

}
```

상기한 로직에 대한 검증은 이 유닛테스트로 시행하였다. 

MultipartFile 객체를 직접 만드는 것은 의미도 없고 귀찮다고 생각하여 Mockito를 사용해 모의 객체를 생성하였다. .getOriginalFilename() 만 구현하면 되기 때문에 유닛테스트 작성이 무척 용이해졌다. 

.csv, .xlsx 두 가지 파일이름을 가진 파일 업로드 요청이 정상적으로 분기 처리되는지를 검증했다. 

ArticleListView.kt

```kotlin
data class ArticleListView(

        @field:JsonProperty("title")
        val title: String? = null,

        @field:JsonProperty("group")
        val group: String? = null,

        @field:JsonProperty("href")
        val link: String? = null) {

    companion object {
        fun fromModel(article: Article, link: String): ArticleListView {
            return ArticleListView(title = article.title, link = link)
        }
    }
}
```

DTO는 내 취향대로 코틀린으로 작성하였고 덕분에 보일러플레이트를 상당 부분 줄일 수 있었다. 

기본값으로 null을 부여한 것은 com.fasterxml.jackson 의 요구사항 중 하나인 기본 생성자 구현 조건을 맞춰주기 위한 것으로 코틀린을 이용한 DTO 구현의 유일한 군더더기이다. (사실 이 부분도 jackson-kotlin-plugin을 통해 해결할 수 있다.)

**프론트엔드**

사실 프론트엔드 구현은 이번 프로젝트의 주안점이 아니라고 생각되어 너무 프론트에서 힘빼지 않으려고 노력했다.

그래도 하다보니 뭔가 쪼리는 느낌(?) 이 들어서 프론트에도 엄청난 공을 들이는 내 자신을 발견하게 됐다. 

로그인 기능을 구현했고, 상태 관리 도구도 붙이려고 mobx를 유력히 검토했으나 시간 관계상 그러지 못했다. 전반적으로 프론트엔드 코드는 매우 지저분하고 즉각적인 리팩토링을 요하는 상태다. 

프로젝트의 요구조건 중 하나인 REST API 제공과 간단한 프론트엔드 구현을 모두 충족하는 방향으로 개발하려면 React.js 기반의 SPA 제작이 제격인 것 같아 그렇게 구현했다. 

NewArticlePage.js

```javascript
    render() {

        if(this.state.submitFinished) {
            return <Redirect to="/"/>
        }

        return(
            <div className="articlewrite-container">
                <h3 className="articlewrite-pagetitle">새 공지사항</h3>
                <GroupCheckbox groups={this.state.availableGroups} onCheckPressed={this.onGroupCheckChanged.bind(this)} title={this.onTitleChange}/>
                <div className="articlewrite-editor-container" ref={ref => this.editorContainerRef = ref}>
                    
                </div>
                <div className="articlewrite-wordcount">
                    <h3 className="word-count">{this.state.wordCount}/1000</h3>
                </div>
                <div className="articlewrite-fileupload-container">
                    <div className="fileupload-header">
                        <h3 className="fileupload-title">파일 업로드</h3>
                        <p className="fileupload-caption">파일은 5MB 이하의 xlsx, xls, txt, csv, tsv만 업로드하실 수 있습니다.</p>
                    </div>
                    <div className="fileupload-component">
                        <DropZone accept={this.fileTypes} onDrop={this.onFileDrop} className="fileupload-dropzone">
                        &nbsp;
                            <div className="dropzone-element fileupload-dropzone-caption">파일을 이곳에 드래그 앤 드롭 하세요.</div>
                            <button className="dropzone-element fileupload-dropzone-click">클릭해서 파일 탐색</button> 
                        </DropZone>
                    </div>
                    <UploadedFiles files={this.state.uploadedFiles}/>
                </div>
                <div className="articlewrite-submit">
                    <button className="btn btn-success" onClick={this.onSubmitClick}>공지사항 등록</button>
                </div>
            </div>
        )
    }
```

새로운 게시글을 올리는 smart component의 렌더 부분만 가져와봤다. 프론트엔드는 대체로 이런 수준의 구현이다. 

child component 몇 개를 더 분리해낼 수 있는 지점이 보이지만 마크업 / UI 제작 -> 컴포넌트 분리 -> 로직 작성 의 flow를 따라가다보니 시간 관계상 컴포넌트 분리를 제대로 할 수 없었다. 

![단체사진](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/40226538-e25f0e5e-5ac6-11e8-954a-0a083551b3d2.JPG)

다들 수고가 많았다.

**멘토님의 총평** 

멘토님은 다음과 같은 평을 남겨주셨다. 

- (멘토님 질문) 몽고DB를 사용한 이유가 궁금하다.
    - (나의 답변) 사용자 알림이라는 데이터 자체가 JOIN 등 관계형 데이터베이스의 특성으로 보관할 이유가 없다고 보고 NoSQL을 도입해봄.
    - 학습 목적으로 MongoDB를 실 프로젝트에 한 번 붙여보았다(진짜 멍청한 대답이었다고 생각된다)

- 사실 알림 기능을 훨씬 단순히 구현할 수 있었을 텐데 별도의 저장소를 사용한 것이 신기하게 느껴진다(들으면서 식은땀이 줄줄 흘렀다).

- (인텔리제이 REST Client 기능) .http 파일은 인텔리제이 플러그인인가? Postman을 사용하지 않는가? 
- 서버 배포를 옵션 요구사항으로 포함한 것은 리눅스를 다루는 모습을 보기 위함이었다. 

이 정도의 말씀을 여기에 옮길 수 있을 것 같다. 

**우리 주제에 대한 나의 소고**

우리 멘토님은 멘티들에게 프로젝트 진행에 있어 극도의 자유도를 부여하셨고 우리도 적극적으로 자유를 만끽하며 각자 하고싶은 대로 프로젝트를 구현했다. 셋 다 다른 버전의 스프링을 사용했고 어떤 분은 Spring @MVC, 어떤 분은 Spring Boot를 사용했다. 데이터 persistence에 있어서도 MyBatis를 사용한 분도 있고 Spring Data JPA를 사용하신 분도 있다. 

약간의 부가기능이 추가된 CRUD 프로젝트를 진행하며 **의외로 많은 평가의 포인트가 있었을 수도 있겠다** 는 생각이 문득 들었다. 프론트 구현 능력은 어느정도가 되는지, 객체지향적 설계는 어느 정도 할 줄 아는지 등을 볼 수 있었을 것 같고 멘토님께서 직접 말씀하신 것처럼 배포 과제를 통해 리눅스환경 친숙도도 평가해볼만 할 것 같다. 

한 가지 아쉬운 점은 "네이버 쇼핑" 이라는 사업분야와 직접적인 연관관계를 떠올릴 만한 주제는 아니었다고 생각되고, 예비 개발자들의 이목을 잡아끌 수 있는 흥미로운 주제는 아니었다는 생각이 들었다. 물론 다른팀 주제들이 좀 말도 안되게 재밌는 것들이 많아서 상대적으로 그래보인 부분도 있다. 

나같은 경우 커머스, 광고 분야에 관심이 있고 향후 개발자로써의 커리어를 만드는데 있어 주 도메인을 커머스사업, 혹은 물류/모빌리티쪽으로 잡아나갈 생각이기에 내게 이번 주제는 의미도 있고 또 네이버 쇼핑을 만드는 개발자와 1박 2일간 같은 공간에서 호흡할 수 있는 기적같은 기회이기도 했다. *다른 친구들에겐 그정도까진 아니었을 것이다.* 

---

#### 마무리

1박 2일이 금방 지나갔다. 개인적으로는 너무나 큰 의미가 있는 시간이었다. 

작년(2017년) 1월 깃허브에 첫 커밋, 이후 개인 프로젝트를 통해 개발에 흥미를 얻고 같은해 8월에 개발 공부 본격적 시작, 그리고 오늘 네이버 커넥트원에 서기까지의 시간들이 주마등처럼 흘러갔다. 물론 하나의 과정에 지나지 않는 거지만 다시 한 번 좋은 선생님들과 함께 **기적같은 한 해를 보냈다**는 생각이 들어 마음이 너무 벅찼다. 

향후 네이버 인턴 기회로까지 이어지면 좋겠지만 결과에 관계 없이 당당하게 멋진 도전을 했다는 생각이 들어 내 자신이 대견하게 느껴졌다. 무엇보다 나의 개발자로써의 진로설정을 반신반의하시던 부모님께 어쨌든 국내 1위 IT기업의 행사 참가라는 가시적인 결과물을 보여드릴 수 있어 기쁘게 생각한다. 

또 앞으로 무엇을 공부해야 할지에 대해 힌트를 얻을 수 있는 시간들이었다. **데 이 터 베 이 스** 를 지금보다 훨씬 더 잘 알아야겠다는 생각이 많이 들었다. 지금도 완전 모르는 것은 아니지만 **오히려 지금같은 상태가 더 위험할 수도 있겠다**는 생각이 문득 들었다. 

신선한 충격과 자극을 쉴새없이 떨어트려 준 1박 2일이었다!

![핵데이](https://s3.ap-northeast-2.amazonaws.com/techblog-static-imgs/40225954-7008e4fc-5ac5-11e8-915a-cb0504803d55.JPG)


즐거운 기회를 만들어주신 네이버 임직원 여러분들과 고수아 멘토님, 그리고 수고해주신 모든 분들께 깊이 감사드립니다.

**프로젝트 리포지토리 보러가기**

- [프론트엔드](https://github.com/seulgiwendy/hackday-shopping-frontend)
- [백엔드](https://github.com/seulgiwendy/hackday-shopping)

---

**기술면접에 가게 됐다.** 곧 면접 후기도 쓸게요. 

*내가 네이버 면접이라니 ㅜㅜㅜㅜㅜㅜ*
















