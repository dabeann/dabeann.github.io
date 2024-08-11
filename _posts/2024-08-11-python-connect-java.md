---
layout: post
title: Java로 Python 코드 실행하기
subtitle: Java <-> Python
categories: SpringBoot
tags: [python]
---

## Java에서 Python코드를 실행한 이유
구현하려는 기능 중에, LangChain을 이용해 대화 분석이 필요한 기능이 있었다.  
LangChain을 Java로 쓴 예시는 하나도 나오지 않았기에, LangChain은 Python으로 코드를 작성하고 그 Python코드를 Java코드를 통해 
실행하자! 라는 생각을 하게 되었다.  
(**절대** 하지 말길...실행 시간이 매우 느리다.)

## Jython
- Jython은 파이썬 2.7 문법을 따른다.
- 현재 내 파이썬 버전은 3.12 이다.
- Python을 다운그레이드하는 것은 좋지 않은 방법 같아 Jython을 과감히 포기했다.

## ProcessBuilder
나는 컨트롤러에서 파이썬 파일을 실행한 후 결과를 받아왔다.

### Controller
- try 구문 안쪽만 보면 된다.
- ProcessBuilder의 첫 번째 인자는 파이썬 경로이고, 두 번째 인자는 실행할 python 파일 경로이다. 세 번째 인자는 python파일에서 요구하는 파라미터이다.
- python 파일의 실행 결과를 output이란 String으로 만든다. return값, print값 모두 포함되어 하나의 String으로 만들어진다.
- 실행 결과인 output을 serivce단에서 잘 가공해서 사용한다.  
  
- 한글 설정을 하지 않으면 UTF-16LE로 파이썬 코드가 리턴되어 글자가 깨진다.
    - BufferedReader 부분에 꼭 `StandardCharsets.UTF_8`를 추가해주자.

```java
@GetMapping("/end/{counselId}")
    ResponseEntity<CommonResponse<CounselResultResponse>> endCounsel(@PathVariable("counselId") @NotNull Long counselId) {
        if (counselService.isFinishedCounsel(counselId)) {
            return new ResponseEntity<>(new CommonResponse<>("종료된 상담 클릭",
                    counselService.getFinishCounselWithDialog(counselId)), HttpStatus.OK);
        } else {
            counselService.validateBeforeEndCounsel(counselId);
            try {

                ProcessBuilder processBuilder = new ProcessBuilder("/usr/bin/python3",
                        "src/main/java/com/backend/dorandoran/counsel/service/python/EndCounsel.py",
                        String.valueOf(counselId));
                processBuilder.redirectErrorStream(true);

                Process process = processBuilder.start();
                BufferedReader reader = new BufferedReader(
                        new InputStreamReader(process.getInputStream(), StandardCharsets.UTF_8));
                StringBuilder output = new StringBuilder();
                String line;

                while ((line = reader.readLine()) != null) {
                    output.append(line);
                    output.append(System.lineSeparator());
                }

                int exitCode = process.waitFor();
                if (exitCode != 0) {
                    return new ResponseEntity<>(
                            new CommonResponse<>("Error: Python script execution failed with exit code "
                                    + exitCode + "\n" + output.toString().trim(), null), HttpStatus.BAD_REQUEST);
                }

                String resultWithSummary = output.toString().trim();

                return new ResponseEntity<>(new CommonResponse<>("상담 결과",
                        counselService.endCounsel(counselId, resultWithSummary)), HttpStatus.OK);
            } catch (Exception e) {
                StringWriter sw = new StringWriter();
                PrintWriter pw = new PrintWriter(sw);
                e.printStackTrace(pw);
                String stackTrace = sw.toString();
                return new ResponseEntity<>(new CommonResponse<>("Error: " + stackTrace, null),
                        HttpStatus.BAD_REQUEST);
            }
        }
    }
```

### Python
- 이전 대화 내역을 요약하고, 점수를 추출하는 기능을 한다.
- 이전에는 LangChain을 사용했었는데, LangChain을 제거하고 전체 대화를 GPT에게 통째로 보내도록 수정하였다.

```python
# -*- coding: utf-8 -*-
import sys
#print(sys.path)

import warnings

warnings.filterwarnings("ignore", category=DeprecationWarning)

import os
from dotenv import load_dotenv
import traceback
import psycopg2
from psycopg2.extras import RealDictCursor
import openai
from langchain_openai import OpenAI

load_dotenv()

openai.api_key = os.getenv('OPENAI_API_KEY')
sys.stdout.reconfigure(encoding='utf-8')

llm = OpenAI(
    temperature=0,
    openai_api_key=openai.api_key,
    model_name=os.getenv('MODEL_NAME')
)


def generate_chat_summary(counsel_id):
    # PostgreSQL 데이터베이스 연결 설정
    conn = psycopg2.connect(
        user=os.getenv('DATASOURCE_USERNAME'),
        password=os.getenv('DATASOURCE_PASSWORD'),
        host=os.getenv('DATASOURCE_HOST'),
        port=os.getenv('DATASOURCE_PORT'),
        dbname=os.getenv('DATASOURCE_DBNAME')
    )
    cursor = conn.cursor(cursor_factory=RealDictCursor)

    try:
        # 이전 대화 내역 조회
        cursor.execute("""
            SELECT role, contents FROM dialog
            WHERE counsel_id = %s
            ORDER BY created_date_time ASC
        """, (counsel_id,))
        previous_conversations = cursor.fetchall()

        # 전체 대화 내역을 history로 저장
        history = [
            {"role": conv["role"], "content": conv["contents"]} for conv in previous_conversations
        ]
        messages = [{"role": "system", "content": "다음은 내담자와 상담자의 대화입니다. "
                                                  "이 대화의 내용을 150글자 이내로 간단히 요약해주세요.:" + str(history)}]

        # GPT 모델에게 요약 요청
        response = openai.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            max_tokens=150,
            temperature=0.7,
            n=1,
            stop=None
        )

        summary = response.choices[0].message.content

        # counsel 테이블의 summary 컬럼 업데이트
        cursor.execute("""
            UPDATE counsel
            SET summary = %s
            WHERE counsel_id = %s
        """, (summary, counsel_id))

        conn.commit()

        # 심리점수 내기
        messages = [
            {"role": "system", "content": "대화 내역을 바탕으로 우울, 스트레스, 불안 점수를 추출합니다."},
            {"role": "assistant", "content": "1,-2,3 형태로 추출합니다. 점수는 -3~3점 중 정수로 추출합니다. 상태가 호전된다면 양수, 악화된다면 음수로 추출합니다."},
            {"role": "user", "content": f"{str(history)}를 기반으로 점수를 추출합니다. 우울, 스트레스, 불안 점수를 주어진 형식에 맞게 정확히 추출합니다."}
        ]

        # GPT 모델에게 점수 추출 요구
        # 프롬포트 수정 필요
        response = openai.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            max_tokens=150,
            temperature=0.7,
            n=1,
            stop=None
        ).choices[0].message.content
        print(response)

        return summary

    except Exception as e:
        error_message = "Error: {}\n{}".format(e, traceback.format_exc())
        print(error_message)
        conn.rollback()
        return str(error_message)

    finally:
        cursor.close()
        conn.close()

def main(counsel_id):
    response = generate_chat_summary(counsel_id)
    print(response)

if __name__ == "__main__":
    counsel_id = sys.argv[1]
    main(counsel_id)
```

### 모듈 설치
- 파이썬을 실행하기 위해선 필요한 모듈을 설치해야 한다.
- 테스트는 먼저 모두 local에서 했기에 다음과 같이 모듈을 설치한다.  
  
```bash
cd C:\Users\dabin\AppData\Local\Programs\Python\Python312\Lib\site-packages
pip install langchain-openai
```
- langchain-openai외에도 필요한 모듈을 설치한다.

## Error
내가 ProcessBuilder를 이용해서 Java에서 Python코드를 실행하는 것을 만드는 동안 겪은 에러를 나열해 보았다.

### 파이썬 모듈 설치 경로
맨 처음에는 로컬에서 실행하면 서버 파이썬으로 돌아가는 줄 알고(바보다)  
ec2 서버에 계속 모듈 설치하고 왜 안되지? 반복했다.  
  
오랜 검색 끝에 내 로컬 파이썬 모듈 패키지 경로  
```
C:\Users\dabin\AppData\Local\Programs\Python\Python312\Lib\site-packages
```  
를 찾게 되었다.  
  
서버에 올리게 되면 서버 파이썬에도 패키지를 설치해야 한다.

### 한글 깨짐 현상
처음엔 아무 설정을 안했더니 
![image](https://github.com/user-attachments/assets/8fe1d776-542e-454b-89ef-fa4e742fd7d7)
이런 식으로 한글이 전부 깨져서 리턴됐다.  
오른쪽 아래 text 글씨를 보면 리턴되는 값이 UTF-16LE임을 알 수 있다.  
  
UTF-8로 변환하기 위해 Python에 
```python
# -*- coding: utf-8 -*-
```
위 코드도 써보고 다양한 방법을 시도했지만 변화는 없었다.  
  
Python을 실행하는 **Java**코드에서 한글 설정을 적용해야 했다.  
위에도 적었지만, 
```java
BufferedReader reader = new BufferedReader(
                        new InputStreamReader(process.getInputStream(), StandardCharsets.UTF_8));
```
BufferedReader를 생성할 때 꼭 UTF-8로 설정해줘야 한다.  

### .env 파일 설정
파이썬 파일과 같은 경로에 .env파일을 둬야 한다.  
application.yml과 같은 경로에 뒀더니 인식을 못한다.

### Server와 Local에서의 차이
ProcessBuilder 부분이 다르다. 아래는 ec2서버에서 되는 코드이다.
```java
ProcessBuilder processBuilder = new ProcessBuilder("/usr/bin/python3",
                    "src/main/java/com/backend/dorandoran/counsel/service/python/EndCounsel.py",
                    String.valueOf(counselId));
```
  
로컬에서 실행하려면 "/usr/bin/python3" -> "python"으로 바꾸면 된다.  
  
또한 **엔터**를 String으로 받게 되면  
로컬에서는 **"\\\r\\\n"**이다.  
서버에서는 **"\\\n"**이다.  
String split하는 코드가 있다면 주의해야 한다. 이건 내가 겪은 String 문제이기에 다른 것이 또 있을 수 있다.

## 후기
- 프로젝트를 할 때 언어는 하나만 사용하자.
- 파이썬 코드를 실행하는 api를 요청하면 짧게는 5초 길게는 10초 정도 나왔다. (매우 오래걸린다.)
  - 간단한 a + b 같은 코드라면 괜찮겠지만 나는 GPT api를 호출해서 더 느리다.
- 뭐든 연결하면 좋을 줄 알았지만 아니었다. 그래도 좋은 경험이었다.
- 에러 처리가 불편해지고, output으로 오는 String 처리에서도 에러가 많이 났다.
- 서버와 로컬에서의 실행 결과 String이 다를 수 있다는 단점이 있다.

---
[참고 블로그](https://www.delftstack.com/ko/howto/java/call-python-script-from-java-code/)