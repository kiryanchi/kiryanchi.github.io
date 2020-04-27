---
layout: post
title: Firebase 에러 - Uncaught exception in Firebase runloop (3.0.0)
date: '2020-04-12'
category: java
tags: ['android', 'java']
permalink: '/blog/:categories/:title'
---

Android Studio 에서 Firebase 를 연동하고 난 뒤, 테스트겸 실행을 해보는데 자꾸 에러가 떴다.

```
Uncaught exception in Firebase runloop (3.0.0). Please report to support@firebase.com
```

구글링을 해보니 여러 해결법이 있었는데 다 소용없었고 이걸로 해결됐다.

app 수준의 build.gradle 에서

```gradle
// 기존
implementation 'com.google.firebase:firebase-database:16.0.1'
// 수정
implementation 'com.google.firebase:firebase-database:17.0.0'
```

위에서 아래로 수정해주니 정상 작동했다.