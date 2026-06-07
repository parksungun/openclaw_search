# 자바 SMTP 메일 전송

## 예제

```java
// 1. SMTP 서버 설정 (Gmail 기준, 다른 메일 서비스는 host/port 달라짐)
Properties props = new Properties();
props.put("mail.smtp.host", "smtp.gmail.com");          // SMTP 서버 주소
props.put("mail.smtp.port", "587");                      // TLS 포트 (465는 SSL)
props.put("mail.smtp.auth", "true");                     // 인증 필요
props.put("mail.smtp.starttls.enable", "true");          // TLS 활성화

// 2. 세션 생성 (인증 정보 포함)
Session session = Session.getInstance(props, new javax.mail.Authenticator() {
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication("당신의이메일@gmail.com", "앱비밀번호"); // 앱 비밀번호 사용
    }
});

// 3. 이메일 메시지 작성
Message message = new MimeMessage(session);
message.setFrom(new InternetAddress("당신의이메일@gmail.com"));        // 발신자
message.setRecipients(Message.RecipientType.TO, "수신자@example.com"); // 수신자
message.setSubject("제목: 자바 SMTP 테스트");                           // 제목
message.setText("안녕하세요! 자바로 보낸 첫 번째 SMTP 메일입니다.");    // 본문

// 4. 이메일 전송
Transport.send(message); // 세션 정보로 자동 전송

System.out.println("이메일이 성공적으로 전송되었습니다!");
```

## 설명

- **Properties 설정**: `mail.smtp.host`, `port`, `auth`, `starttls.enable`는 필수
  - Gmail은 TLS(587) 또는 SSL(465) 지원 → 보통 587+TLS 권장
  - 다른 제공자: Naver( smtp.naver.com:587 ), Outlook( smtp-mail.outlook.com:587 )

- **Session + Authenticator**: `PasswordAuthentication`에 이메일 주소 + **앱 비밀번호** 입력
  - 일반 비밀번호 대신 **2단계 인증 후 발급한 앱 비밀번호** 사용 필수 (Gmail 등 대부분)
  - 앱 비밀번호 발급: 계정 설정 → 보안 → 2단계 인증 → 앱 비밀번호 생성

- **Message 설정**:
  - `setFrom()`, `setRecipients()`, `setSubject()`, `setText()` 또는 `setContent()`로 구성
  - HTML 내용은 `message.setContent(htmlContent, "text/html; charset=utf-8")` 사용

- **Transport.send()**: Session에 설정된 SMTP 설정으로 자동 전송

## 함께 학습하면 좋은 것

- JavaMail API 기본 구조
- Gmail 앱 비밀번호 발급 및 설정
- HTML 이메일 전송 (MimeMultipart 사용)
- 첨부파일 추가 방법 (MimeBodyPart + FileDataSource)
- 예외 처리 및 전송 실패 로그 수집
- SMTP 서버 별 설정 차이 (Gmail, Naver, Outlook 등)
