---
layout: post
title: Java 21 Visitor Pattern - 로직과 구조의 분리
description: 자바 21 패턴 매칭을 활용한 Visitor Pattern 구현
date:   2025-08-29 09:40:00 +0900
author: Jeongjin Kim
categories: Java
tags:	Java Visitor Pattern
---

Java에서 객체 타입별로 다른 처리가 필요할 때 instanceof 남발로 인한 복잡한 코드와 확장성 문제를 **Visitor Pattern**으로 해결하는 방법을 다룹니다.

음악 스트리밍 서비스 예제를 통해 패턴의 원리를 단계별로 설명하고, Java 21의 최신 기능들과 실무 적용 방법까지 다룹니다.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

---

오늘도 평범한(?) 목요일 아침, 커피를 마시며 코드를 보던 중 이런 생각이 들었습니다.

"아, 또 `instanceof` 지옥이군..."

```java
public void processShape(Shape shape) {
    if (shape instanceof Circle circle) {
        // 원 처리 로직
    } else if (shape instanceof Rectangle rectangle) {
        // 사각형 처리 로직
    } else if (shape instanceof Triangle triangle) {
        // 삼각형 처리 로직
    }
    // 새로운 도형이 추가될 때마다 이 메소드를 수정해야 함...
}
```

이런 코드를 본 적 있나요? 저는 이런 패턴을 볼 때마다 마음이 조금 무거워집니다. 왜냐하면 새로운 도형이나 처리 방식이 추가될 때마다 기존 코드를 건드려야 하기 때문입니다.

그런데 이런 문제를 해결하는 아주 간지나는 패턴이 있습니다. 바로 **Visitor Pattern**이라고 하는 녀석입니다.


## Visitor Pattern이 뭐지?

Visitor Pattern은 실제 로직을 가지고 있는 객체(Visitor)가 로직을 적용할 객체(Element)를 방문하면서 실행하는 패턴입니다. 쉽게 말하면 **로직과 구조를 분리**하는 패턴이에요.

오너쉐프 레스토랑을 생각해봅시다. 여러 고객(Element)들이 있고, 오너쉐프(Visitor)가 각 고객에게 주문을 받습니다. 고객은 그냥 고객일 뿐이고, 오너쉐프가 "어떻게 음식를 만들지"라는 로직을 가지고 있죠.

## 문제 상황: 음악 스트리밍 서비스

이번엔 조금 다른 예시로 접근해보겠습니다. 음악 스트리밍 서비스를 개발한다고 해볼게요.

우리 서비스에는 여러 종류의 음악 콘텐츠가 있습니다:
- 일반 곡 (Song)
- 팟캐스트 (Podcast)
- 라이브 방송 (LiveStream)

그리고 이 콘텐츠들에 대해 다양한 작업을 수행해야 합니다:
- 재생 통계 분석
- 추천 알고리즘 처리
- 음성 인식 및 자막 생성
- 저작권 검증

자, 이걸 어떻게 설계하시겠어요?

## 첫 번째 시도: 직접 구현

```java
public interface MusicContent {
    void generateStats();
    void processRecommendation();
    void generateSubtitles();
    void verifyLicense();
}

public class Song implements MusicContent {
    private String title;
    private String artist;
    private Duration duration;
    
    @Override
    public void generateStats() {
        System.out.println("노래 " + title + "의 재생 통계를 생성합니다.");
    }
    
    @Override
    public void processRecommendation() {
        System.out.println("노래 기반 추천을 처리합니다.");
    }
    
    @Override
    public void generateSubtitles() {
        System.out.println("가사 자막을 생성합니다.");
    }
    
    @Override
    public void verifyLicense() {
        System.out.println("음악 저작권을 확인합니다.");
    }
}

public class Podcast implements MusicContent {
    private String title;
    private String host;
    private String topic;
    
    @Override
    public void generateStats() {
        System.out.println("팟캐스트 " + title + "의 재생 통계를 생성합니다.");
    }
    
    @Override
    public void processRecommendation() {
        System.out.println("토픽 기반 추천을 처리합니다.");
    }
    
    @Override
    public void generateSubtitles() {
        System.out.println("음성 인식으로 자막을 생성합니다.");
    }
    
    @Override
    public void verifyLicense() {
        System.out.println("팟캐스트 저작권을 확인합니다.");
    }
}
```

음... 동작은 하지만 뭔가 어색합니다. `Song` 클래스가 왜 추천 알고리즘을 알아야 하죠? `Podcast` 클래스가 왜 자막 생성 로직을 알아야 할까요?

**문제점:**
1. **단일 책임 원칙 위배**: 각 콘텐츠 클래스가 너무 많은 책임을 가짐
2. **확장성 문제**: 새로운 처리 로직이 추가되면 모든 클래스를 수정해야 함
3. **유지보수성**: 추천 알고리즘이 바뀌면 모든 콘텐츠 클래스를 건드려야 함

## 두 번째 시도: 로직 분리

그럼 이렇게 해보면 어떨까요?

```java
public interface MusicContent {
    String getTitle();
    // 기본 정보만 가지고 있음
}

public class Song implements MusicContent {
    private String title;
    private String artist;
    private Duration duration;
    
    public String getTitle() { return title; }
    public String getArtist() { return artist; }
    public Duration getDuration() { return duration; }
}

public class MusicProcessor {
    public void generateStats(MusicContent content) {
        if (content instanceof Song song) {
            System.out.println("노래 " + song.getTitle() + "의 재생 통계를 생성합니다.");
        } else if (content instanceof Podcast podcast) {
            System.out.println("팟캐스트 " + podcast.getTitle() + "의 재생 통계를 생성합니다.");
        }
        // instanceof 체크가 계속 늘어남...
    }
    
    public void processRecommendation(MusicContent content) {
        if (content instanceof Song song) {
            System.out.println("노래 기반 추천을 처리합니다.");
        } else if (content instanceof Podcast podcast) {
            System.out.println("토픽 기반 추천을 처리합니다.");
        }
        // 또 instanceof...
    }
}
```

음... 조금 나아졌지만 여전히 문제가 있네요.

**문제점:**
1. **instanceof 남발**: 새로운 콘텐츠 타입이 추가되면 모든 메소드에 분기 추가
2. **컴파일러의 도움을 받지 못함**: 새 콘텐츠 타입을 추가했는데 처리 로직을 빼먹어도 컴파일러가 알려주지 않음
3. **코드 중복**: 비슷한 구조의 분기문이 반복됨

## Java 21의 Pattern Matching 시도

Java 21에서는 패턴 매칭이 많이 발전했으니까, 이걸로 해결해볼까요?

```java
public class MusicProcessor {
    public void generateStats(MusicContent content) {
        switch (content) {
            case Song song -> 
                System.out.println("노래 " + song.getTitle() + "의 재생 통계를 생성합니다.");
            case Podcast podcast -> 
                System.out.println("팟캐스트 " + podcast.getTitle() + "의 재생 통계를 생성합니다.");
            case LiveStream stream -> 
                System.out.println("라이브 방송의 실시간 통계를 생성합니다.");
        }
    }
}
```

오, 이건 좀 깔끔하네요! 하지만 **여전히 새로운 콘텐츠 타입이 추가되면 모든 처리 메소드를 수정**해야 합니다.

## Visitor Pattern으로 해결

이제 Visitor Pattern을 적용해서 앞선 문제를 해결해 보겠습니다.

### 1단계: Visitor 인터페이스 정의

```java
public interface MusicContentVisitor {
    void visit(Song song);
    void visit(Podcast podcast);
    void visit(LiveStream liveStream);
}
```

### 2단계: Element 인터페이스 정의

```java
public interface MusicContent {
    void accept(MusicContentVisitor visitor);
    String getTitle();
}
```

### 3단계: 구체적인 Element 구현

```java
public class Song implements MusicContent {
    private String title;
    private String artist;
    private Duration duration;
    
    public Song(String title, String artist, Duration duration) {
        this.title = title;
        this.artist = artist;
        this.duration = duration;
    }
    
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);  // 핵심: 자기 자신을 visitor에게 전달
    }
    
    // getters
    public String getTitle() { return title; }
    public String getArtist() { return artist; }
    public Duration getDuration() { return duration; }
}

public class Podcast implements MusicContent {
    private String title;
    private String host;
    private String topic;
    private Duration duration;
    
    public Podcast(String title, String host, String topic, Duration duration) {
        this.title = title;
        this.host = host;
        this.topic = topic;
        this.duration = duration;
    }
    
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
    
    // getters
    public String getTitle() { return title; }
    public String getHost() { return host; }
    public String getTopic() { return topic; }
    public Duration getDuration() { return duration; }
}

public class LiveStream implements MusicContent {
    private String title;
    private String streamer;
    private boolean isLive;
    
    public LiveStream(String title, String streamer, boolean isLive) {
        this.title = title;
        this.streamer = streamer;
        this.isLive = isLive;
    }
    
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
    
    // getters
    public String getTitle() { return title; }
    public String getStreamer() { return streamer; }
    public boolean isLive() { return isLive; }
}
```

### 4단계: 구체적인 Visitor 구현

이제 각 기능별로 Visitor를 구현해봅시다.

```java
public class StatisticsVisitor implements MusicContentVisitor {
    private int totalViews = 0;
    private Duration totalDuration = Duration.ZERO;
    
    @Override
    public void visit(Song song) {
        System.out.println("🎵 노래 '" + song.getTitle() + "' by " + song.getArtist() + "의 통계를 분석합니다.");
        totalDuration = totalDuration.plus(song.getDuration());
        totalViews++;
    }
    
    @Override
    public void visit(Podcast podcast) {
        System.out.println("🎙️ 팟캐스트 '" + podcast.getTitle() + "' (" + podcast.getTopic() + ")의 통계를 분석합니다.");
        totalDuration = totalDuration.plus(podcast.getDuration());
        totalViews++;
    }
    
    @Override
    public void visit(LiveStream liveStream) {
        System.out.println("📡 라이브 방송 '" + liveStream.getTitle() + "'의 실시간 통계를 수집합니다.");
        if (liveStream.isLive()) {
            System.out.println("   현재 방송 중입니다!");
        }
        totalViews++;
    }
    
    public void printSummary() {
        System.out.println("\n📊 통계 요약:");
        System.out.println("총 컨텐츠 수: " + totalViews);
        System.out.println("총 재생 시간: " + totalDuration.toMinutes() + "분");
    }
}

public class RecommendationVisitor implements MusicContentVisitor {
    private List<String> recommendations = new ArrayList<>();
    
    @Override
    public void visit(Song song) {
        System.out.println("🎯 '" + song.getTitle() + "' 기반 음악 추천을 생성합니다.");
        recommendations.add("'" + song.getArtist() + "'의 다른 곡들");
        recommendations.add("유사한 장르의 인기곡");
    }
    
    @Override
    public void visit(Podcast podcast) {
        System.out.println("🎯 '" + podcast.getTopic() + "' 주제 기반 팟캐스트 추천을 생성합니다.");
        recommendations.add("'" + podcast.getTopic() + "' 관련 다른 에피소드");
        recommendations.add("'" + podcast.getHost() + "'의 다른 팟캐스트");
    }
    
    @Override
    public void visit(LiveStream liveStream) {
        System.out.println("🎯 '" + liveStream.getStreamer() + "' 스트리머 기반 추천을 생성합니다.");
        recommendations.add("'" + liveStream.getStreamer() + "'의 다른 방송");
        recommendations.add("유사한 스트리밍 채널");
    }
    
    public List<String> getRecommendations() {
        return new ArrayList<>(recommendations);
    }
}

public class SubtitleVisitor implements MusicContentVisitor {
    @Override
    public void visit(Song song) {
        System.out.println("🎤 '" + song.getTitle() + "'의 가사 자막을 생성합니다.");
        System.out.println("   가사 데이터베이스에서 동기화 중...");
    }
    
    @Override
    public void visit(Podcast podcast) {
        System.out.println("🗣️ 팟캐스트 '" + podcast.getTitle() + "'의 음성인식 자막을 생성합니다.");
        System.out.println("   AI 음성인식 엔진 가동 중...");
    }
    
    @Override
    public void visit(LiveStream liveStream) {
        if (liveStream.isLive()) {
            System.out.println("📺 라이브 방송의 실시간 자막을 생성합니다.");
            System.out.println("   실시간 음성인식 처리 중...");
        } else {
            System.out.println("📺 녹화된 방송의 자막을 생성합니다.");
        }
    }
}
```

### 5단계: 사용해보기

```java
public class MusicStreamingService {
    public static void main(String[] args) {
        // 다양한 콘텐츠 생성
        var musicContents = List.of(
            new Song("Shape of You", "Ed Sheeran", Duration.ofMinutes(4)),
            new Podcast("Tech Talk", "John Doe", "AI Technology", Duration.ofMinutes(45)),
            new LiveStream("Gaming Night", "ProGamer123", true),
            new Song("Blinding Lights", "The Weeknd", Duration.ofMinutes(3)),
            new LiveStream("Music Jam Session", "MusicMaster", false)
        );
        
        // 통계 분석
        System.out.println("=== 통계 분석 ===");
        var statsVisitor = new StatisticsVisitor();
        for (var content : musicContents) {
            content.accept(statsVisitor);
        }
        statsVisitor.printSummary();
        
        System.out.println("\n=== 추천 생성 ===");
        var recommendationVisitor = new RecommendationVisitor();
        for (var content : musicContents) {
            content.accept(recommendationVisitor);
        }
        
        System.out.println("\n🎯 추천 목록:");
        recommendationVisitor.getRecommendations().forEach(rec -> 
            System.out.println("  - " + rec));
        
        System.out.println("\n=== 자막 생성 ===");
        var subtitleVisitor = new SubtitleVisitor();
        for (var content : musicContents) {
            content.accept(subtitleVisitor);
        }
    }
}
```

## Java 21의 Modern Features 활용

Java 21에서는 여러 현대적 기능들을 활용해서 Visitor Pattern을 더욱 깔끔하게 만들 수 있습니다.

### Records와 Sealed Classes 활용

```java
// Sealed interface로 타입을 제한
public sealed interface MusicContent 
    permits Song, Podcast, LiveStream {
    void accept(MusicContentVisitor visitor);
    String getTitle();
}

// Records로 간결한 데이터 클래스
public record Song(
    String title,
    String artist,
    Duration duration
) implements MusicContent {
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
}

public record Podcast(
    String title,
    String host,
    String topic,
    Duration duration
) implements MusicContent {
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
}

public record LiveStream(
    String title,
    String streamer,
    boolean isLive
) implements MusicContent {
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
}
```

## 실제 사용 사례와 확장

### 새로운 콘텐츠 타입 추가

새로운 콘텐츠 타입인 `AudioBook`을 추가해봅시다.

```java
public record AudioBook(
    String title,
    String author,
    String narrator,
    Duration duration,
    int chapterCount
) implements MusicContent {
    @Override
    public void accept(MusicContentVisitor visitor) {
        visitor.visit(this);
    }
}
```

기존의 모든 Visitor 인터페이스에 메소드를 추가해야 합니다:

```java
public interface MusicContentVisitor {
    void visit(Song song);
    void visit(Podcast podcast);
    void visit(LiveStream liveStream);
    void visit(AudioBook audioBook); // 새로 추가
}
```

그리고 모든 구현체에서 이 메소드를 구현해야 합니다. 컴파일러가 강제하기 때문에 빠뜨릴 염려가 없죠!

### 새로운 기능 추가

저작권 검증 기능을 추가해봅시다.

```java
public class LicenseVisitor implements MusicContentVisitor {
    private final Map<String, String> licenseInfo = new HashMap<>();
    
    @Override
    public void visit(Song song) {
        System.out.println("🔒 '" + song.title() + "'의 음악 저작권을 확인합니다.");
        // 음원 저작권 DB 조회 로직
        licenseInfo.put(song.title(), "KOMCA-2024-001");
    }
    
    @Override
    public void visit(Podcast podcast) {
        System.out.println("🔒 팟캐스트 '" + podcast.title() + "'의 저작권을 확인합니다.");
        // 팟캐스트 저작권 확인 로직
        licenseInfo.put(podcast.title(), "PODCAST-LICENSE-VALID");
    }
    
    @Override
    public void visit(LiveStream liveStream) {
        System.out.println("🔒 라이브 방송의 실시간 저작권을 모니터링합니다.");
        // 실시간 저작권 모니터링
        licenseInfo.put(liveStream.title(), "REALTIME-MONITORING");
    }
    
    @Override
    public void visit(AudioBook audioBook) {
        System.out.println("🔒 오디오북 '" + audioBook.title() + "'의 출판 권리를 확인합니다.");
        licenseInfo.put(audioBook.title(), "PUBLISHING-RIGHT-OK");
    }
    
    public Map<String, String> getLicenseInfo() {
        return Map.copyOf(licenseInfo);
    }
}
```

기존 콘텐츠 클래스들은 전혀 수정하지 않았지만, 새로운 기능이 완벽하게 동작합니다!

## Visitor Pattern의 장단점

### 장점들

**1. 확장성**: 새로운 동작을 기존 객체 구조 수정 없이 추가 가능

```java
// 새로운 기능 추가도 이렇게 간단!
public class ExportVisitor implements MusicContentVisitor {
    // JSON, XML, CSV 등으로 내보내기
}
```

**2. 단일 책임 원칙**: 각 Visitor가 특정 작업에만 집중

**3. 개방-폐쇄 원칙**: 기존 코드는 닫혀있고, 확장에는 열려있음

**4. 타입 안정성**: 컴파일러가 모든 케이스 구현을 강제

### 단점들

**1. 복잡성**: 단순한 경우에는 과도할 수 있음

**2. Element 구조 변경의 어려움**: 새로운 Element 타입 추가 시 모든 Visitor 수정 필요

**3. 캡슐화 깨짐**: Element 내부 상태를 Visitor에 노출해야 할 수 있음

## 언제 사용하면 좋을까?

### 적합한 상황

1. **Element 구조가 안정적**: 콘텐츠 타입이 자주 변하지 않는 경우
2. **다양한 알고리즘**: 같은 객체들에 대해 다양한 처리가 필요한 경우
3. **관련 없는 동작들**: 서로 다른 성격의 작업들을 분리하고 싶은 경우

### 부적합한 상황

1. **자주 변하는 구조**: Element 타입이 자주 추가/제거되는 경우
2. **단순한 처리**: 간단한 작업에는 오버엔지니어링
3. **성능이 중요한 경우**: Double dispatch로 인한 오버헤드


## 실전 팁

### 1. Default 메소드 활용

```java
public interface MusicContentVisitor {
    void visit(Song song);
    void visit(Podcast podcast);
    void visit(LiveStream liveStream);
    void visit(AudioBook audioBook);
    
    // 기본 구현 제공
    default void beforeVisit(MusicContent content) {
        System.out.println("처리 시작: " + content.getTitle());
    }
    
    default void afterVisit(MusicContent content) {
        System.out.println("처리 완료: " + content.getTitle());
    }
}
```

### 2. Builder 패턴과 조합

```java
public class VisitorChain {
    private final List<MusicContentVisitor> visitors = new ArrayList<>();
    
    public static VisitorChain builder() {
        return new VisitorChain();
    }
    
    public VisitorChain addStatistics() {
        visitors.add(new StatisticsVisitor());
        return this;
    }
    
    public VisitorChain addRecommendation() {
        visitors.add(new RecommendationVisitor());
        return this;
    }
    
    public VisitorChain addSubtitle() {
        visitors.add(new SubtitleVisitor());
        return this;
    }
    
    public void process(List<MusicContent> contents) {
        for (var visitor : visitors) {
            System.out.println("\n=== " + visitor.getClass().getSimpleName() + " 실행 ===");
            contents.forEach(content -> {
                visitor.beforeVisit(content);
                content.accept(visitor);
                visitor.afterVisit(content);
            });
        }
    }
}

// 사용법
VisitorChain.builder()
    .addStatistics()
    .addRecommendation()
    .addSubtitle()
    .process(musicContents);
```

### 3. Exception Handling

```java
public abstract class SafeVisitor implements MusicContentVisitor {
    
    protected abstract void doVisit(Song song) throws Exception;
    protected abstract void doVisit(Podcast podcast) throws Exception;
    protected abstract void doVisit(LiveStream liveStream) throws Exception;
    protected abstract void doVisit(AudioBook audioBook) throws Exception;
    
    @Override
    public final void visit(Song song) {
        try {
            doVisit(song);
        } catch (Exception e) {
            handleException("Song", song.title(), e);
        }
    }
    
    @Override
    public final void visit(Podcast podcast) {
        try {
            doVisit(podcast);
        } catch (Exception e) {
            handleException("Podcast", podcast.title(), e);
        }
    }
    
    @Override
    public final void visit(LiveStream liveStream) {
        try {
            doVisit(liveStream);
        } catch (Exception e) {
            handleException("LiveStream", liveStream.title(), e);
        }
    }
    
    @Override
    public final void visit(AudioBook audioBook) {
        try {
            doVisit(audioBook);
        } catch (Exception e) {
            handleException("AudioBook", audioBook.title(), e);
        }
    }
    
    protected void handleException(String type, String title, Exception e) {
        System.err.println("❌ " + type + " '" + title + "' 처리 중 오류: " + e.getMessage());
    }
}

// 사용 예시
public class RobustStatisticsVisitor extends SafeVisitor {
    private int successCount = 0;
    private int errorCount = 0;
    
    @Override
    protected void doVisit(Song song) throws Exception {
        if (song.duration().isNegative()) {
            throw new IllegalArgumentException("음수 재생시간");
        }
        System.out.println("✅ 노래 통계 처리: " + song.title());
        successCount++;
    }
    
    @Override
    protected void doVisit(Podcast podcast) throws Exception {
        if (podcast.topic().isBlank()) {
            throw new IllegalArgumentException("주제가 없음");
        }
        System.out.println("✅ 팟캐스트 통계 처리: " + podcast.title());
        successCount++;
    }
    
    @Override
    protected void doVisit(LiveStream liveStream) throws Exception {
        System.out.println("✅ 라이브 스트림 통계 처리: " + liveStream.title());
        successCount++;
    }
    
    @Override
    protected void doVisit(AudioBook audioBook) throws Exception {
        System.out.println("✅ 오디오북 통계 처리: " + audioBook.title());
        successCount++;
    }
    
    @Override
    protected void handleException(String type, String title, Exception e) {
        super.handleException(type, title, e);
        errorCount++;
    }
    
    public void printReport() {
        System.out.println("\n📊 처리 결과:");
        System.out.println("성공: " + successCount + "건");
        System.out.println("실패: " + errorCount + "건");
    }
}
```

### 4. Generic Visitor Pattern

```java
public interface GenericVisitor<T> {
    T visit(Song song);
    T visit(Podcast podcast);
    T visit(LiveStream liveStream);
    T visit(AudioBook audioBook);
}

public interface GenericElement {
    <T> T accept(GenericVisitor<T> visitor);
}

// 사용 예시: 크기 계산 Visitor
public class SizeCalculatorVisitor implements GenericVisitor<Long> {
    @Override
    public Long visit(Song song) {
        return song.duration().toSeconds() * 128; // 대략적인 파일 크기 (KB)
    }
    
    @Override
    public Long visit(Podcast podcast) {
        return podcast.duration().toSeconds() * 64; // 낮은 품질
    }
    
    @Override
    public Long visit(LiveStream liveStream) {
        return liveStream.isLive() ? -1L : 0L; // 라이브는 크기 측정 불가
    }
    
    @Override
    public Long visit(AudioBook audioBook) {
        return audioBook.duration().toSeconds() * 32; // 음성 전용
    }
}
```

## 마무리

Visitor Pattern은 처음엔 복잡해 보일 수 있지만, 한번 이해하고 나면 정말 강력한 도구입니다. 

특히 Java 21의 현대적 기능들(Records, Sealed Classes, Virtual Threads, Pattern Matching)과 함께 사용하면 더욱 빛이 납니다.

**핵심을 다시 정리하면:**

1. **Double Dispatch**: 런타임에 두 객체의 타입을 모두 고려한 메소드 호출
2. **관심사 분리**: 객체 구조와 처리 로직의 완전한 분리  
3. **확장성**: 기존 코드 수정 없이 새로운 기능 추가
4. **타입 안전성**: 컴파일러의 도움으로 실수 방지

모든 곳에 Visitor Pattern을 적용할 필요는 없습니다. 객체 구조가 안정적이고, 다양한 처리 로직이 필요한 경우에만 사용해야합니다.

**"패턴을 위한 패턴이 아니라, 문제를 해결하기 위한 패턴"**이라는 것을 항상 기억하시길 바랍니다.

지금은 AI에게 자유자재로 코드를 만들라고 시킬수 있을 만큼 기본기가 중요하다고 생각합니다. 우리 모두 화이팅입니다.🎉



<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>