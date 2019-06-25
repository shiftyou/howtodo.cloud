---
layout: post
title:  Async로 호출하여 Sync로 응답 받기
categories: java
tags: java sync async
---

# Async 로 호출하여 Sync 로 응답받기

기존에 몇가지 조건 때문에 aysnc 호출을 sync 처럼 응답받아야만 하는 경우가 있었다.  
이에 대해서 어떻게 구현하면 되는지 기존에 만들었던 Oracle Chatbot에서 kakao talk 연동하는 모듈로 기록을 남겨본다.


# 시나리오

시나리오는 다음과 같다.

![Alt text](https://monosnap.com/image/t8kl6BRZdJWXHV5zZ38Wm6WJYQTKZw)

1. 클라이언트가 프락시(현재 만들고자 하는 서비스)를 호출한다. 클라이언트는 sync 방식으로 응답받는다.
1. 프락시는 대상서비스를 async 방식으로 호출한다.
1. 프락시는 async 방식으로 대상서비스에서 응답이 오기를 기다린다.
1. 프락시는 타임아웃을 두어 응답이 올때 까지 무한정 기다리는 것을 방지한다.
1. 프락시는 응답이 오면 클라이언트에게 응답을 보낸다.

즉, async 방식의 서비스를 sync 방식으로 호출하기 위한 코드이다.

이를 위한 코드는 Oracle Chatbot 에서 Kakao 의 API를 호출하는 내용으로 설명한다.
>source : https://github.com/jonggyoukim/kakaoconnector-for-chatbot

현재 Kakao의 서비스가 변경이 되어 해당 방식으로 호출을 할 수는 없지만, async 서비스를 sync 방식으로 호출하기 위한 방법은 다른 목적일 때에도 동일하게 사용될 수 있다.


# 클라이언트에서 sync 방식으로 호출되는 메소드


~~~java

private BlockingQueue<String> getQueue(String threadId) {
    BlockingQueue<String> queue = null;

    logger.info("/message threadid = " + threadId);
    if (!queueMap.containsKey(threadId)) {
        BlockingQueue<String> _queue = new ArrayBlockingQueue<>(1);
        queueMap.put(threadId, _queue);
        queue = _queue;
    } else {
        queue = queueMap.get(threadId);
    }
    queue.clear();
    return queue;
}

public String message(@RequestHeader Map<String, Object> headers, @RequestBody Map<String, Object> bodyMap) {

    try {
        // 카카오로 보낼 메시지를 만든다. - payload
        String payload = makePayload(bodyMap);

        // userId - threadId - queue 를 연결한다.
        String threadId = Thread.currentThread().getName();
        userMap.put(userId, threadId);
        logger.info("/message 사용자[" + userId + "]를 셋팅했습니다.");
        BlockingQueue<String> queue = getQueue(threadId);

        // 오라클 봇으로 연결
        RestTemplate restTemplate = new RestTemplate();
        HttpEntity<String> entity = new HttpEntity<String>(payload, getHeaders(payload));
        logger.info("/message entry[" + entity.toString() + "]");
        ResponseEntity<String> responseEntity = restTemplate.exchange(botUri, HttpMethod.POST, entity,
                String.class);
        logger.info("/message exchanged");

        if (responseEntity.getStatusCode().is2xxSuccessful()) {
            logger.info("/message 200 return");
            try {
                do {
                    // 응답이 올 때 까지 기다림 4.2초
                    long t1 = System.currentTimeMillis();

                    response = queue.poll(wait_time, TimeUnit.MILLISECONDS);

                    long t2 = System.currentTimeMillis();
                    logger.info("응답시간(타임아웃:5초) : " + df.format(t2 - t1));

                    // 받은 응답 체크
                    if (response == null) {
                        kakao = new KAKAO("서버의 응답이 느려서 대답할 수가없어요!!");
                        // response = "{ \"message\":{ \"text\" : \"서버의 응답이
                        // 느려서 대답할
                        // 수가없어요!!\" } }";
                        logger.info("/message response = null");
                    } else {
                        kakao = convertBotToKakao(response);

                        logger.info("/message response=[" + response + "]");
                    }
                    logger.info("/message kakao=[" + kakao + "]");
                    logger.info("/message kakako.chunked=[" + kakao.isChunked() + "]");

                    if (kakao.isChunked()) {
                        if (responseMap.containsKey(userId)) {
                            KAKAO _kakao = responseMap.get(userId);
                            _kakao.merge(kakao);
                            kakao = _kakao;

                            responseMap.put(userId, kakao);
                        } else {
                            responseMap.put(userId, kakao);
                        }
                        logger.info("/message 다음메시지가 있어 [" + userId + "]를 삭제 하지 않겠습니다.");
                    } else {
                        if (responseMap.containsKey(userId)) {
                            KAKAO _kakao = responseMap.remove(userId);
                            _kakao.merge(kakao);
                            kakao = _kakao;
                            logger.info("/message merged kakao=[" + kakao + "]");

                            userMap.remove(userId);
                        } else {
                            userMap.remove(userId);
                        }
                        logger.info("/kakao [" + userId + "]를 삭제하였습니다.");
                    }
                    logger.info("/message responseHash.size = " + responseMap.size());
                    logger.info("/message replyHash.size = " + userMap.size());
                } while (kakao.isChunked());
            } catch (InterruptedException e) {
                e.printStackTrace();
                userMap.remove(userId);
            }
        } // bot으로 request 성공일 때

    } catch (Exception e) {
        e.printStackTrace();
        kakao = new KAKAO(e.getMessage());
        userMap.remove(userId);
    }
}
~~~

# 타겟서비스에서 async 방식으로 응답을 위한 호출받는 메소드

fromWebhook 메소드는 다른 서버로 부터 reply가 async로 오는 것을 받는 메소드이다.
~~~java
String fromWebhook(@RequestBody Map<String, Object> bodyMap) throws JsonProcessingException {
    logger.info("/kakao request=[" + bodyMap + "]");

    String userId = mapper.convertValue(bodyMap.get("userId"), String.class);
    String payload = mapper.writeValueAsString(bodyMap);

    if (!userMap.containsKey(userId)) {
        logger.info("/kakao +-----------------------------------------+");
        logger.info("/kakao | 사용자[" + userId + "]가 없습니다.            ");
        logger.info("/kakao +-----------------------------------------+");
    } else {
        String threadId = userMap.get(userId);
        @SuppressWarnings("unchecked")
        BlockingQueue<String> queue = queueMap.get(threadId);
        try {
            queue.offer(payload, 5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    return "ok";
}
~~~

중요한 것은 BlockingQueue를 사용하여 대기하고 있다가 응답이 오면 반응하는 형식이다.  
코드만 보아도 이해하리라 믿는다!!

혹시, 이해 안 되시면 질문 달아주시면 설명드립니다.

