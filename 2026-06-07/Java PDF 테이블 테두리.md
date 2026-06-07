# Java PDF 테이블 테두리

## 예제

### Maven 의존성

```xml
<!-- Apache PDFBox 의존성을 추가하면 Java 코드로 PDF 페이지, 글자, 선, 사각형을 직접 그릴 수 있다. -->
<dependency><!-- Maven 의존성 하나를 선언한다. -->
  <!-- PDFBox가 속한 Maven groupId를 지정한다. -->
  <groupId>org.apache.pdfbox</groupId><!-- Apache PDFBox 라이브러리 그룹을 지정한다. -->
  <!-- PDFBox 핵심 모듈 artifactId를 지정한다. -->
  <artifactId>pdfbox</artifactId><!-- PDF 생성과 그리기에 필요한 핵심 모듈을 지정한다. -->
  <!-- 기존 Java PDF 생성 문서와 같은 PDFBox 3.x 버전을 사용한다. -->
  <version>3.0.7</version><!-- PDFBox 3.0.7 버전을 사용한다. -->
</dependency><!-- Maven 의존성 선언을 끝낸다. -->
```

### Java 코드

```java
import java.awt.Color; // 테이블 헤더 배경색과 선 색상을 지정하기 위해 Color 클래스를 가져온다.
import java.io.File; // PDF 결과 파일 경로를 표현하기 위해 File 클래스를 가져온다.
import java.io.IOException; // PDF 생성 중 발생할 수 있는 입출력 예외를 처리하기 위해 IOException 클래스를 가져온다.
import java.nio.file.Path; // 폰트 파일 경로를 안전하게 표현하기 위해 Path 클래스를 가져온다.
import java.nio.file.Paths; // 문자열 경로를 Path 객체로 바꾸기 위해 Paths 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDDocument; // PDF 문서 전체를 만들고 저장하기 위해 PDDocument 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDPage; // PDF에 들어갈 한 페이지를 만들기 위해 PDPage 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.PDPageContentStream; // 페이지에 선, 사각형, 글자를 그리기 위해 PDPageContentStream 클래스를 가져온다.
import org.apache.pdfbox.pdmodel.font.PDType0Font; // 한글을 PDF에 출력하기 위해 유니코드 TTF 폰트를 로드하는 클래스를 가져온다.
public class PdfTableBorderExample { // PDF에 테두리가 있는 테이블을 그리는 예제 클래스를 선언한다.
    public static void main(String[] args) throws IOException { // PDF 생성 과정의 IOException을 호출자에게 전달하는 main 메서드를 선언한다.
        Path fontPath = Paths.get("fonts/NotoSansKR-Regular.ttf"); // 프로젝트 fonts 폴더의 한글 폰트 파일 경로를 준비한다.
        File outputFile = new File("build/java-pdf-table-border.pdf"); // 생성된 PDF를 저장할 파일 경로를 준비한다.
        outputFile.getParentFile().mkdirs(); // build 폴더가 없을 때 PDF 저장 전에 폴더를 만든다.
        try (PDDocument document = new PDDocument()) { // 작업이 끝나면 자동으로 닫히는 새 PDF 문서를 만든다.
            PDPage page = new PDPage(); // 테이블을 그릴 빈 PDF 페이지를 만든다.
            document.addPage(page); // 만든 페이지를 PDF 문서에 추가한다.
            PDType0Font font = PDType0Font.load(document, fontPath.toFile()); // 한글이 깨지지 않도록 TTF 폰트를 문서에 로드한다.
            try (PDPageContentStream stream = new PDPageContentStream(document, page)) { // 페이지에 실제 내용을 그릴 스트림을 열고 자동으로 닫히게 한다.
                stream.beginText(); // 제목을 쓰기 위해 텍스트 모드를 시작한다.
                stream.setFont(font, 16); // 제목에 사용할 폰트와 크기를 설정한다.
                stream.newLineAtOffset(72, 730); // 페이지 왼쪽에서 72포인트, 아래에서 730포인트 위치로 이동한다.
                stream.showText("주문 목록"); // PDF 상단에 테이블 제목을 출력한다.
                stream.endText(); // 제목 텍스트 쓰기 모드를 종료한다.
                String[] headers = {"상품명", "수량", "단가", "금액"}; // 테이블 헤더에 표시할 컬럼명을 준비한다.
                String[][] rows = {{"키보드", "2", "35,000", "70,000"}, {"마우스", "1", "18,000", "18,000"}, {"모니터", "1", "220,000", "220,000"}}; // 테이블 본문에 표시할 데이터를 준비한다.
                float tableX = 72; // 테이블의 왼쪽 시작 좌표를 지정한다.
                float tableY = 690; // 테이블의 위쪽 시작 좌표를 지정한다.
                float rowHeight = 28; // 각 행의 높이를 포인트 단위로 지정한다.
                float[] colWidths = {180, 70, 100, 100}; // 각 컬럼의 너비를 포인트 단위로 지정한다.
                drawHeader(stream, font, headers, tableX, tableY, rowHeight, colWidths); // 헤더 행의 배경, 테두리, 텍스트를 그린다.
                for (int rowIndex = 0; rowIndex < rows.length; rowIndex++) { // 본문 데이터 행 개수만큼 반복한다.
                    float rowY = tableY - rowHeight * (rowIndex + 1); // 현재 본문 행의 위쪽 y 좌표를 계산한다.
                    drawRow(stream, font, rows[rowIndex], tableX, rowY, rowHeight, colWidths); // 현재 행의 셀 테두리와 텍스트를 그린다.
                } // 본문 행 반복을 끝낸다.
            } // 페이지 내용 스트림을 닫아서 그린 내용을 문서에 반영한다.
            document.save(outputFile); // 완성된 PDF 문서를 파일로 저장한다.
        } // PDF 문서를 닫아서 내부 리소스를 정리한다.
    } // main 메서드를 끝낸다.
    private static void drawHeader(PDPageContentStream stream, PDType0Font font, String[] headers, float x, float y, float rowHeight, float[] colWidths) throws IOException { // 헤더 행을 그리는 메서드를 선언한다.
        float currentX = x; // 첫 번째 헤더 셀의 왼쪽 x 좌표를 준비한다.
        for (int colIndex = 0; colIndex < headers.length; colIndex++) { // 헤더 컬럼 수만큼 반복한다.
            drawCell(stream, font, headers[colIndex], currentX, y, colWidths[colIndex], rowHeight, true); // 현재 헤더 셀의 배경, 테두리, 텍스트를 그린다.
            currentX += colWidths[colIndex]; // 다음 셀의 왼쪽 좌표로 이동한다.
        } // 헤더 컬럼 반복을 끝낸다.
    } // drawHeader 메서드를 끝낸다.
    private static void drawRow(PDPageContentStream stream, PDType0Font font, String[] row, float x, float y, float rowHeight, float[] colWidths) throws IOException { // 본문 행을 그리는 메서드를 선언한다.
        float currentX = x; // 첫 번째 본문 셀의 왼쪽 x 좌표를 준비한다.
        for (int colIndex = 0; colIndex < row.length; colIndex++) { // 본문 컬럼 수만큼 반복한다.
            drawCell(stream, font, row[colIndex], currentX, y, colWidths[colIndex], rowHeight, false); // 현재 본문 셀의 테두리와 텍스트를 그린다.
            currentX += colWidths[colIndex]; // 다음 셀의 왼쪽 좌표로 이동한다.
        } // 본문 컬럼 반복을 끝낸다.
    } // drawRow 메서드를 끝낸다.
    private static void drawCell(PDPageContentStream stream, PDType0Font font, String text, float x, float y, float width, float height, boolean header) throws IOException { // 하나의 셀을 그리는 공통 메서드를 선언한다.
        if (header) { // 현재 셀이 헤더인지 확인한다.
            stream.setNonStrokingColor(new Color(230, 236, 245)); // 헤더 배경을 연한 회색 파란색으로 설정한다.
            stream.addRect(x, y - height, width, height); // 헤더 배경으로 채울 사각형 경로를 만든다.
            stream.fill(); // 방금 만든 사각형 내부를 현재 채우기 색상으로 칠한다.
        } // 헤더 배경 처리 조건문을 끝낸다.
        stream.setStrokingColor(Color.BLACK); // 셀 테두리 선 색상을 검정으로 설정한다.
        stream.setLineWidth(0.8f); // 셀 테두리 두께를 0.8포인트로 설정한다.
        stream.addRect(x, y - height, width, height); // 셀의 바깥 테두리가 될 사각형 경로를 만든다.
        stream.stroke(); // 방금 만든 사각형 경로를 선으로 그려 셀 테두리를 표시한다.
        stream.beginText(); // 셀 안에 글자를 쓰기 위해 텍스트 모드를 시작한다.
        stream.setNonStrokingColor(Color.BLACK); // 셀 텍스트 색상을 검정으로 설정한다.
        stream.setFont(font, header ? 11 : 10); // 헤더는 11포인트, 본문은 10포인트 폰트로 설정한다.
        stream.newLineAtOffset(x + 8, y - 18); // 셀 왼쪽에서 8포인트, 셀 위쪽에서 18포인트 아래 위치로 이동한다.
        stream.showText(text); // 셀에 들어갈 텍스트를 출력한다.
        stream.endText(); // 셀 텍스트 쓰기 모드를 종료한다.
    } // drawCell 메서드를 끝낸다.
} // PdfTableBorderExample 클래스를 끝낸다.
```

## 설명

PDFBox는 HTML 테이블처럼 `table` 태그를 제공하지 않는다. 그래서 표를 만들 때는 **좌표를 계산해서 사각형과 텍스트를 직접 그린다**고 생각하면 된다. 핵심은 각 셀의 `x`, `y`, `width`, `height` 값을 정하고, 그 위치에 `addRect()`와 `stroke()`로 테두리를 그린 뒤, 같은 셀 안쪽에 텍스트를 쓰는 방식이다.

테두리는 `addRect()`와 `stroke()` 조합으로 만든다. `addRect(x, y - height, width, height)`처럼 `y - height`를 쓰는 이유는 PDF 좌표계가 왼쪽 아래를 기준으로 하기 때문이다. 사람이 생각하는 테이블의 `y`는 보통 셀의 위쪽 위치인데, PDFBox의 사각형 시작점은 왼쪽 아래이므로 행 높이만큼 내려간 값을 넣어야 셀이 원하는 위치에 그려진다.

헤더 배경색은 `setNonStrokingColor()`로 채우기 색상을 정하고, `fill()`로 사각형 내부를 칠해서 만든다. 그 뒤 다시 `setStrokingColor()`와 `stroke()`를 사용해 테두리를 그린다. 채우기와 선은 서로 다른 상태라서, 배경을 칠한 뒤에도 테두리 색상과 두께를 따로 지정하는 습관이 좋다.

셀 내부 텍스트는 자동으로 가운데 정렬되거나 줄바꿈되지 않는다. 예제에서는 `x + 8`, `y - 18`처럼 셀 안쪽 여백을 직접 줬다. 업무용 PDF에서는 긴 상품명, 주소, 비고처럼 길이가 달라지는 값이 많으므로 문자열 폭을 계산해 줄바꿈하거나, 폰트 크기를 줄이거나, 컬럼 너비를 넉넉히 잡아야 한다.

자주 하는 실수는 다음과 같다.

- `fill()`로 헤더 배경을 칠한 뒤 `stroke()`를 호출하지 않아 테두리가 사라지는 경우
- PDF 좌표계가 왼쪽 아래 기준이라는 점을 잊고 y 좌표를 반대로 계산하는 경우
- 한글 폰트를 로드하지 않아 셀 텍스트가 깨지는 경우
- 셀 너비보다 긴 텍스트를 그대로 출력해 다음 셀 영역을 침범하는 경우
- 여러 페이지로 넘어가는 긴 테이블인데 페이지 추가와 헤더 반복 처리를 하지 않는 경우

행이 많아질 때는 현재 행의 아래쪽 y 좌표가 페이지 하단 여백보다 작아지는지 검사해야 한다. 작아지면 새 `PDPage`를 만들고, 새 `PDPageContentStream`을 연 뒤, 헤더를 다시 그리고 다음 행부터 이어서 출력하는 방식으로 페이지 나눔을 구현한다.

## 함께 학습하면 좋은 것

- PDFBox 좌표계와 포인트 단위
- `PDPageContentStream`의 `stroke()`, `fill()`, `setLineWidth()` 차이
- 한글 폰트 임베딩과 `PDType0Font`
- 문자열 폭 계산을 통한 셀 안 자동 줄바꿈
- 긴 테이블의 페이지 나눔과 헤더 반복 출력
