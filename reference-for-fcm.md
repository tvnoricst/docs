* https://firebase.google.com/docs/cloud-messaging/send-message#node.js

## payload

### 공통
```js
const message = {
    notification: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#Notification
        title: '',
        body: '',
        image: '', 
    },
    data: { // string: string 형식만 사용
    },
};
```

### 기기별 설정
```js
const message = {
    notification: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#Notification
        title: '',
        body: '',
        image: '', 
    },
    data: { // string: string 형식만 사용
    },
    android: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#androidconfig
        collapse_key: '',
        priority: 'NORMAL|HIGH', // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#AndroidMessagePriority
        ttl: '',
        restricted_package_name: '',
        data: {},
        notification: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#androidnotification
            title: '',
            body: '',
            icon: '',
            color: '',
            sound: '',
            tag: '',
            click_action: '',
            body_loc_key: '',
            body_loc_args: [
                ''
            ],
            title_loc_key: '',
            title_loc_args: [
                ''
            ],
            channel_id: '',
            ticker: '',
            sticky: true|false,
            event_time: '',
            local_only: true|false,
            notification_priority: 'PRIORITY_UNSPECIFIED|PRIORITY_MIN|PRIORITY_LOW|PRIORITY_DEFAULT|PRIORITY_HIGH|PRIORITY_MAX', // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#notificationpriority
            default_sound: true|false,
            default_vibrate_timings: true|false,
            default_light_settings: true|false,
            vibrate_timings: [
                ''
            ],
            visibility: 'VISIBILITY_UNSPECIFIED|PRIVATE|PUBLIC|SECRET', // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#visibility
            notification_count: 0,
            light_settings: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#lightsettings
                color: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#color
                    red: 0,
                    green: 0,
                    blue: 0,
                    alpha: 0
                },
                light_on_duration: '3.5s',
                light_off_duration: '3.5s'
            },
            image: '',
            bypass_proxy_notification: true|false
        },
        fcm_options: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#AndroidFcmOptions
            analytics_label: ''
        },
        direct_boot_ok: true|false
    },
    apns: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#apnsconfig
        headers: { // https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns
        },
        payload: { // https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification
        },
        fcm_options: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#apnsfcmoptions
            analytics_label: '',
            image: ''
        }
    },
    webpush: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#webpushconfig
        headers: {
        },
        notification: { // https://developer.mozilla.org/en-US/docs/Web/API/Notification
        },
        data: { // string: string 형식만 사용
        },
        fcm_options: { // https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#WebpushFcmOptions
            link: '',
            analytics_label: ''
        }
    }
};
```

## Exmple

### 일반 전송 - 단일기기
```js
const message = {
    notification: {
        title: '',
        body: ''
    },
    token: ''
};
//
getMessaging()
    .send(message)
    .then((response) => {
        // response = projects/{project_id}/messages/{message_id}
        console.log('Successfully sent message:', response);
    })
    .catch((error) => {
        console.log('Error sending message:', error);
    });
```

### 일반 전송 - 여러기기
```js
const message = {
    notification: {
        title: '',
        body: ''
    },
    tokens: [] // 호출당 최대 500개
};
//
getMessaging()
    .sendMulticast(message)
    .then((response) => {
        if (response.failureCount > 0) {
            const failedTokens = [];
            response.responses.forEach((resp, idx) => {
                if (!resp.success) {
                    failedTokens.push(registrationTokens[idx]);
                }
            });
            console.log('List of tokens that caused failures: ' + failedTokens);
        }
    });
```

### 일반 전송 - 주제별
```js
const message = {
    notification: {
        title: '',
        body: ''
    },
    topic: 'TopicA'
};
//
getMessaging()
    .send(message)
    .then((response) => {
        // response = projects/{project_id}/messages/{message_id}
        console.log('Successfully sent message:', response);
    })
    .catch((error) => {
        console.log('Error sending message:', error);
    });
```

### 일반 전송 - 주제별 - 조건부
```js
const message = {
    notification: {
        title: '',
        body: ''
    },
    condition: `'TopicA' in topics && ('TopicB' in topics || 'TopicC' in topics)`
};
//
getMessaging()
    .send(message)
    .then((response) => {
        // response = projects/{project_id}/messages/{message_id}
        console.log('Successfully sent message:', response);
    })
    .catch((error) => {
        console.log('Error sending message:', error);
    });
```

### 일괄 전송
```js
const messages = []; // 호출당 최대 500건
messages.push({ notification: { title: '', body: '' }, token: '' });
messages.push({ notification: { title: '', body: '' }, tokens: [] });
messages.push({ notification: { title: '', body: '' }, topic: 'TopicA' });
messages.push({ notification: { title: '', body: '' }, condition: `'TopicA' in topics && ('TopicB' in topics || 'TopicC' in topics)` });
//
getMessaging()
    .sendAll(messages)
    .then((response) => {
        console.log(response.successCount + ' messages were sent successfully');
    })
```

### 주제 구독
```js
// These registration tokens come from the client FCM SDKs.
const registrationTokens = [ // 요청당 1000건 제한
  'YOUR_REGISTRATION_TOKEN_1',
  // ...
  'YOUR_REGISTRATION_TOKEN_n'
];

// Subscribe the devices corresponding to the registration tokens to the
// topic.
getMessaging().subscribeToTopic(registrationTokens, topic)
  .then((response) => {
    // See the MessagingTopicManagementResponse reference documentation
    // for the contents of response.
    console.log('Successfully subscribed to topic:', response);
  })
  .catch((error) => {
    console.log('Error subscribing to topic:', error);
  });
```

### 주제 구독 취소
```js
// These registration tokens come from the client FCM SDKs.
const registrationTokens = [ // 요청당 1000건 제한
  'YOUR_REGISTRATION_TOKEN_1',
  // ...
  'YOUR_REGISTRATION_TOKEN_n'
];

// Unsubscribe the devices corresponding to the registration tokens from
// the topic.
getMessaging().unsubscribeFromTopic(registrationTokens, topic)
  .then((response) => {
    // See the MessagingTopicManagementResponse reference documentation
    // for the contents of response.
    console.log('Successfully unsubscribed from topic:', response);
  })
  .catch((error) => {
    console.log('Error unsubscribing from topic:', error);
  });
```