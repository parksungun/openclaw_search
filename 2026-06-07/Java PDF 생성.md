# Java PDF 생성

## 예제

### Maven 의존성

```xml
<!-- PDFBox 라이브러리를 Maven 프로젝트에 추가한다. -->
<dependency><!-- 하나의 외부 라이브러리 의존성 선언을 시작한다. -->
  <!-- Apache PDFBox의 Maven 그룹 ID를 지정한다. -->
  <groupId>org.apache.pdfbox</groupId><!-- PDFBox가 속한 Maven 그룹을 지정한다. -->
  <!-- PDFBox의 핵심 모듈 artifact ID를 지정한다. -->
  <artifactId>pdfbox</artifactId><!-- PDF 문서 생성, 수정, 읽기에 필요한 핵심 라이브러리를 지정한다. -->
  <!-- 2026-06-07 기준 Apache PDFBox 공식 Getting Started 문서의 최신 3.x 버전을 지정한다. -->
  <version>3.0.7</version><!-- PDFBox 3.0.7 버전을 사용한다. -->
</dependency><!-- 외부 라이브러리 의존성 선언을 끝낸다. -->
```

### Java 코드

```java
import java.io.File; // PDF 파일을 저장할 출력 경로를 다루기 위해 File 클래스를 가져온다.
import java.io.IOException; // PDF 생성 중 발생할 수 있는 입출력 예외를 처리하기 위해 IOException 클래스를 가져온다.
import java.nio.file.Path; // 한글 폰트 파일의 경로를 표현하기 위해 Path 클래스를 가져온다.
import java.nio.file.Paths; // 문자열 경로를 Path 객체로 바꾸기 위해 Paths 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDDocument; // 새 PDF 문서 객체를 만들기 위해 PDDocument 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDPage; // PDF 안에 들어갈 한 페이지를 만들기 위해 PDPage 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDPageContentStream; // 페이지에 글자나 선을 그리기 위해 PDPageContentStream 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.font.PDType0Font; // 한글 같은 유니코드 글꼴을 PDF에 임베딩하기 위해 PDType0Font 클래스를 가져온다.
public class PdfCreateExample { // PDF 생성 예제를 실행할 클래스를 선언한다.
    public static void main(String[] args) throws IOException { // PDF 생성 중 IOException이 생기면 호출자에게 던지는 main 메서드를 선언한다.
        Path fontPath = Paths.get("fonts/NotoSansKR-Regular.ttf"); // 프로젝트의 fonts 폴더에 있는 한글 폰트 파일 경로를 준비한다.
        File outputFile = new File("build/java-pdf-example.pdf"); // 생성된 PDF를 저장할 파일 경로를 준비한다.
        outputFile.getParentFile().mkdirs(); // build 폴더가 없으면 PDF 저장 전에 미리 생성한다.
        try (PDDocument document = new PDDocument()) { // 작업이 끝나면 자동으로 닫히는 새 PDF 문서를 만든다.
            PDPage page = new PDPage(); // PDF에 넣을 빈 페이지를 하나 만든다.
            document.addPage(page); // 만든 페이지를 PDF 문서에 추가한다.
            PDType0Font font = PDType0Font.load(document, fontPath.toFile()); // 한글이 깨지지 않도록 TTF 폰트를 PDF 문서에 로드한다.
            try (PDPageContentStream contentStream = new PDPageContentStream(document, page)) { // 페이지에 내용을 그릴 스트림을 열고 자동으로 닫히게 한다.
                contentStream.beginText(); // 텍스트를 쓰기 위한 텍스트 모드를 시작한다.
                contentStream.setFont(font, 16); // 방금 로드한 한글 폰트를 16포인트 크기로 설정한다.
                contentStream.setLeading(24); // 줄바꿈할 때 다음 줄로 내려갈 간격을 24포인트로 설정한다.
                contentStream.newLineAtOffset(72, 720); // 왼쪽 72포인트, 아래에서 720포인트 위치로 시작 좌표를 옮긴다.
                contentStream.showText("자바에서 PDF 생성하기"); // 첫 번째 줄에 제목 텍스트를 출력한다.
                contentStream.newLine(); // 다음 줄로 이동한다.
                contentStream.showText("Apache PDFBox를 사용하면 코드로 PDF를 만들 수 있다."); // 두 번째 줄에 설명 텍스트를 출력한다.
                contentStream.newLine(); // 다음 줄로 이동한다.
                contentStream.showText("한글은 반드시 한글을 지원하는 TTF/OTF 폰트를 로드해야 한다."); // 세 번째 줄에 한글 폰트 주의사항을 출력한다.
                contentStream.endText(); // 텍스트 쓰기 모드를 종료한다.
            } // 페이지 내용 스트림을 닫아서 그린 내용을 문서에 반영한다.
            document.save(outputFile); // 완성된 PDF 문서를 지정한 파일 경로에 저장한다.
        } // PDF 문서를 닫아서 파일 핸들과 내부 리소스를 정리한다.
    } // main 메서드를 끝낸다.
} // PdfCreateExample 클래스를 끝낸다.
```

## 설명

Java에서 PDF를 생성할 때는 보통 **PDFBox** 또는 **iText**를 많이 쓴다. 단순 생성, 텍스트 출력, 이미지 삽입, 기존 PDF 수정, 텍스트 추출까지 폭넓게 다룰 수 있고 Apache License 2.0인 PDFBox가 학습과 일반 업무 자동화에는 부담이 적다.

PDFBox에서 핵심 객체는 세 가지다.

- `PDDocument`: PDF 파일 전체를 의미한다. 새 문서를 만들거나 저장하고 닫는 책임을 가진다.
- `PDPage`: PDF 안의 한 페이지를 의미한다. 문서에 여러 페이지를 추가할 때 이 객체를 반복해서 만든다.
- `PDPageContentStream`: 특정 페이지 위에 텍스트, 선, 이미지 같은 실제 내용을 그리는 도구다.

왜 `try-with-resources`를 쓰는지가 중요하다. PDF 문서는 내부적으로 파일 핸들, 폰트 데이터, 스트림 리소스를 잡고 있기 때문에 닫지 않으면 파일이 깨지거나 저장이 끝나지 않을 수 있다. `PDDocument`와 `PDPageContentStream`은 작업이 끝나면 반드시 닫아야 한다.

한글 PDF에서 가장 자주 하는 실수는 기본 폰트만 쓰는 것이다. PDFBox의 기본 표준 폰트는 한글을 제대로 표현하지 못한다. 그래서 `PDType0Font.load()`로 `NotoSansKR-Regular.ttf` 같은 한글 지원 폰트를 로드해야 한다. 이렇게 하면 폰트가 PDF에 임베딩되어 다른 PC에서도 한글이 깨질 가능성이 줄어든다.

좌표 체계도 주의해야 한다. PDF 좌표는 일반 화면처럼 왼쪽 위가 아니라 **왼쪽 아래가 기준점**이다. 예제의 `newLineAtOffset(72, 720)`은 왼쪽에서 72포인트, 아래에서 720포인트 위치에 텍스트 시작점을 둔다는 뜻이다. A4/Letter 페이지에서 위쪽에 글자를 쓰려면 y 값을 크게 잡아야 한다.

운영 환경에서 PDF를 만들 때는 다음을 추가로 고려해야 한다.

- 긴 문장은 자동 줄바꿈되지 않으므로 직접 문자열 길이를 계산하거나 레이아웃 라이브러리를 붙여야 한다.
- 표, 복잡한 문단, 헤더/푸터가 많으면 직접 좌표를 찍는 방식보다 템플릿 기반 접근이 편하다.
- 웹 서비스에서 PDF를 내려줄 때는 파일 저장 대신 `ByteArrayOutputStream`에 저장한 뒤 HTTP 응답으로 보내는 방식이 흔하다.
- 라이선스가 중요한 업무라면 iText의 AGPL/상용 라이선스 조건과 PDFBox의 Apache License 2.0 차이를 확인해야 한다.

참고 링크:

- Apache PDFBox 공식 사이트: https://pdfbox.apache.org/
- Apache PDFBox Getting Started: https://pdfbox.apache.org/3.0/getting-started.html
- Maven Central PDFBox: https://repo.maven.apache.org/maven2/org/apache/pdfbox/pdfbox/

## 함께 학습하면 좋은 것

- `try-with-resources`와 Java 리소스 자동 해제
- PDF 좌표계와 포인트 단위
- 한글 폰트 임베딩과 유니코드 폰트 처리
- Spring Boot에서 PDF 파일 다운로드 응답 만들기
- HTML을 PDF로 변환하는 OpenHTMLToPDF, Flying Saucer 같은 대안 라이브러리
