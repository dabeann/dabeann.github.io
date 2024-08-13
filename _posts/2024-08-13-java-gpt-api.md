---
layout: post
title: SpringBoot Java로 Gpt api 연동하기
subtitle: 
categories: SpringBoot
tags: [gpt]
---
## application-gpt.yml
- OpenAI에서 발급받은 key를 넣어준다.
- 나는 직접 학습시킨 모델을 사용했기에 모델명도 넣어주었다.  
  

```yml
OPENAI_API_KEY: asdf
MODEL_NAME: asdf
```

## GptConfig
- TOP_P : 이전 단어들을 바탕으로 생성한 후보 중에서, 누적 확률 분포의 상위 p%에 해당하는 후보들을 선택하는 기법
- MAX_TOKEN : 하나의 요청에서 처리할 수 있는 최대 토큰 수
- TEMPERATURE : 갚이 높을수록 더 다양한 출력을 생성, 값이 낮을수록 보수적인 출력을 생성
- TIME_OUT : API 호출의 타임아웃 기간 설정. 여기서는 3600초(1시간)으로 설정했다.
  
  
```java
@Configuration
@PropertySource("classpath:application-gpt.yml")
public class GptConfig {

    @Value("${OPENAI_API_KEY}")
    private String openaiApiKey;

    public static final double TOP_P = 1.0;

    public static final int MAX_TOKEN = 2000;

    public static final double TEMPERATURE = 1.0;

    public static final Duration TIME_OUT = Duration.ofSeconds(3600);

    @Bean
    public OpenAiService openAiService() {
        return new OpenAiService(openaiApiKey, TIME_OUT);
    }
}
```

## service
- 전체 코드는 복잡하기에 gpt에게 메시지를 보내고 응답받는 부분만 가져왔다.

### Gpt에게 보낼 ChatMessage 생성
- System, Assistant, User 같이 여러 Role로 메시지를 담을 수 있다.
  

```java
private List<ChatMessage> generatedPsychologyScoreChatMessage(String history) {
        List<ChatMessage> messages = new ArrayList<>();

        String systemMessage = "대화 내역을 바탕으로 우울, 스트레스, 불안 점수를 추출합니다.";
        ChatMessage systemMsg = new ChatMessage(ChatMessageRole.SYSTEM.toString().toLowerCase(), systemMessage);
        messages.add(systemMsg);

        String assistantMessage = "1,-2,3 형태로 추출합니다. 점수는 -3~3점 중 정수로 추출합니다. 상태가 호전된다면 양수, 악화된다면 음수로 추출합니다.";
        ChatMessage assistantMsg = new ChatMessage(ChatMessageRole.ASSISTANT.toString().toLowerCase(), assistantMessage);
        messages.add(assistantMsg);

        String userMessage = history + "를 기반으로 점수를 추출합니다. 우울, 스트레스, 불안 점수를 주어진 형식에 맞게 정확히 추출합니다.";
        ChatMessage userMsg = new ChatMessage(ChatMessageRole.USER.toString().toLowerCase(), userMessage);
        messages.add(userMsg);

        return messages;
    }
```

### Gpt에 메시지 보내고 응답 받기
- 위 함수에서 리턴한 것을 파라미터로 넣는다.
- 모델명은 내가 설정한 모델로 해도 되고, 기본적인 gpt-4o로 해도 된다.
  
  
```java
private ChatCompletionResult sendOpenAIRequest4o(List<ChatMessage> prompt) {
        ChatCompletionRequest build = ChatCompletionRequest.builder()
                .messages(prompt)
                .maxTokens(GptConfig.MAX_TOKEN)
                .temperature(GptConfig.TEMPERATURE)
                .topP(GptConfig.TOP_P)
                .model("gpt-4o")
                .build();

        return openAiService.createChatCompletion(build);
    }
```

### 위 두 함수 사용
- 아래처럼 써서 gpt의 응답 값을 String으로 받아온다.
- 기능에 맞도록 가공하고 저장하면 된다.
  

```java
List<ChatMessage> chatMessages = generatedPsychologyScoreChatMessage(history);
String stringScores = sendOpenAIRequest4o(chatMessages).getChoices().get(0).getMessage().getContent();
```

### 전체 코드
[service 전체 코드](https://github.com/dabeann/dorandoran/blob/main/src/main/java/com/backend/dorandoran/counsel/service/CounselResultService.java)