# KakaoWork Bot

nodeJS, Express를 이용한 카카오워크 봇 개발 방법

## Express

-   설치

```bash
npm i express -g
npm i express-generator -g
```

-   프로젝트 생성

```bash
# 템플릿 엔진을 pug로 설정
express --view=pug testBot
```

## KakaoWork Bot

다른 사람이 만든 Bot은 물론, 워크스페이스 참가자가 직접 개발할 수 있다

#### [Block Kit](https://www.kakaowork.com/block-kit-builder)

    -   단순하지만 강력한 인터랙션을 사용자가 경험할 수 있도록 제공하는 UI 프레임워크
    -   다양한 블록들을 조합하여 사용
    -   미리 제공되는 정해진 틀 안에서만 항목들을 조합하여 메세지 생성 가능

#### 알림형 커스텀 봇 프로세스

1. Bot 생성
2. Bot 인증
3. 멤버 조회
4. 채팅방 생성
5. 알림형 대화

![알림형프로세스](https://grm-project-template-bucket.s3.ap-northeast-2.amazonaws.com/lesson/les_UVjaD_1618361283021/e73e3bb0f0a29b7fdd9def17eda96d9a8a5d1a9ea5c7cd40654623cdf34ec20f.png)

#### 반응형 커스텀 봇 프로세스 (submit_action)

1. 메세지 전송 (Button Block: submit_action)
2. 액션 버튼 클릭
3. 액션 이벤트 전달 (Callback URL)
4. 액션 이벤트에 따른 다음 시나리오 진행

![반응형프로세스](https://grm-project-template-bucket.s3.ap-northeast-2.amazonaws.com/lesson/les_YSJKa_1618361283004/eb341887a21fd911aaf65ab83fa4c713b48f2cd55b7a7a6b01b4d30c0e1dd3e9.png)

#### 반응형 커스텀 봇 프로세스 (call_modal)

1. 메세지 전송 (Button Block: submit_action)
2. 액션 버튼 클릭
3. 액션 이벤트 전달 (Request URL)
4. Modal 블록 정보 전송
5. Modal 블록 정보 표시
6. 정보 입력 & 액션 버튼 클릭
7. Modal 이벤트 전달 (Callback URL)
8. Modal 종료

![반응형콜프로세스](https://grm-project-template-bucket.s3.ap-northeast-2.amazonaws.com/lesson/les_REdMe_1618361283025/9cb3083a78e319aabf0bebfad72b3c8ce4ae1482a98c591da75fc0d3bda15d42.png)

## Develop

-   baseURL과 headers를 미리 등록한 axios instance

```js
// libs/kakaoWork/index.js
const Config = require("config");

const axios = require("axios");
const kakaoInstance = axios.create({
    baseURL: "https://api.kakaowork.com",
    headers: {
        Authorization: `Bearer ${Config.keys.kakaoWork.bot}`,
    },
});
```

-   유저 목록 검색

```js
exports.getUserList = async () => {
    const res = await kakaoInstance.get("/v1/users.list");
    return res.data.users;
};
```

-   채팅방 생성

```js
exports.openConversations = async ({ userId }) => {
    const data = {
        user_id: userId,
    };
    const res = await kakaoInstance.post("/v1/conversations.open", data);
    return res.data.conversation;
};
```

-   메세지 전송

```js
exports.sendMessage = async ({ conversationId, text, blocks }) => {
    const data = {
        conversation_id: conversationId,
        text,
        ...(blocks && { blocks }),
    };
    const res = await kakaoInstance.post("/v1/messages.send", data);
    return res.data.message;
};
```

-   Root URL 방문 시에 API 호출

```js
// routes/index.js
const express = require("express");
const router = express.Router();

const libKakaoWork = require("../libs/kakaoWork");

router.get("/", async (req, res, next) => {
    // 유저 목록 검색 (1)
    const users = await libKakaoWork.getUserList();

    // 검색된 모든 유저에게 각각 채팅방 생성 (2)
    const conversations = await Promise.all(
        users.map((user) => libKakaoWork.openConversations({ userId: user.id }))
    );

    // 생성된 채팅방에 메세지 전송 (3)
    const messages = await Promise.all([
        conversations.map((conversation) =>
            libKakaoWork.sendMessage({
                conversationId: conversation.id,
                text: "TEST TEXT",
                blocks: [
                    {
                        type: "header",
                        text: "test",
                        style: "blue",
                    },
                    ...
                ],
            })
        ),
    ]);

    res.json({
        users,
        conversations,
        messages,
    });
});

module.exports = router;
```
