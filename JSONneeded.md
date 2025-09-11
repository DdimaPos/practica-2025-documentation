# API Documentation

[Figma](https://www.figma.com/design/Ula9H2lV0QYa53KHYXx1Hq/Student-Forum?node-id=45-1663&p=f&t=1yxBfuh3M22F8p27-0)

## Pool (POST request)
Should be discussed if we will allow to attach files.
```json
{
  "author": "",
  "question": "",
  "response_variants": ["", "", ...],
  "is_anonymous": false,
  "tags": "",
  "files": (?)
}
```

Response:
(?)

---

## Post (POST request)
```json
{
  "author": "",
  "title": "",
  "content": "",
  "is_anonymous": false,
  "tags": "",
  "files": (?)
}
```
Response:
(?)

---

## Edit Post (UPDATE request)
In both cases (edit and delte) we need author value to be sure that we delete right post.
```json
{
  "id": 1,
  "author": "",
  "title": "",
  "content": ""
}
```
Response:
(?)

---

## Delete Post (DELETE request)
```json
{
  "id": 1,
  "author": ""
}
```
Response:
(?)

---

## Post \[parent] (GET request)
Response:
```json
{
  "id": 1,
  "author": "",
  "title": "",
  "content": "",
  "created_at": "2025-09-05T12:00:00Z",
  "rating": "",
  "photo": (?),
  "n_comments": 0
}
```

---

## Replies to post (GET request)
First we will get just an finite amount of replies, it can be extended if user clicks the button.

Response:
```json
[
  {
    "id": 1,
    "author": "",
    "content": "",
    "created_at": "2025-09-05T12:00:00Z",
    "rating": "",
    "photo": (?),
    "n_comments": 0,
    "replies": [ ... ] // same structure
  }
]
```

---

## Upvote / Downvote  (UPDATE request)
If upvote is false => it's downvote
```json
{
  "id": 1,
  "upvote": false
}
```
Response:
(?)

---

## Reply (POST request)
If post is anonymous => replies are also anonymous => fields like author can be empty
```json
{
  "author": "",
  "title": "",
  "content": "",
  "files": (?)
}
```

Response:
(?)

---

## User Registration (POST request)
Here I don't cover verification of email.
```json
{
  "email": "",
  "username": "",
  "name_surname": "",
  "password": "",
  "photo": ""
}
```
Response:
(?)

---

## User Login  (POST request)
```json
{
  "username": "",
  "password": ""
}
```
Response:
(?)

---

## Profile \[main info] (GET request) 
Response:
```json
{
  "id": 1,
  "username": "",
  "name_surname": "",
  "created_at": "2025-09-05T12:00:00Z",
  "rating": "",
  "photo": (?),
  "n_posts": 0,
  "is_verified": (?)
}
```
---

## Profile \[user's posts] (GET request)
Response:
```json
[
  {
    "id": 1,
    "author": "", (?)
    "title": "",
    "content": "",
    "created_at": "2025-09-05T12:00:00Z",
    "rating": "",
    "photo": (?)
  },
  ...
]
```

---


## Most Popular Channels (GET request)
Response:
```json
[
  {
    "id": 1,
    "name": "",
    "photo": "",
    "n_participants": 0
  }
]
```
---


## Most Active Users (GET request)
Response:
```json
[
  {
    "id": 1,
    "name": "",
    "photo": "",
    "n_posts": 0
  }
]
```

---

## Posts (GET request)

Response:
```json
[
  {
    "id": 1,
    "author": "",
    "title": "",
    "content": "",
    "created_at": "2025-09-05T12:00:00Z",
    "rating": "",
    "photo": (?)
  }
]
```


---

## Search  (POST request)
Response is written above.
```json
{
  "content": "",
  "category": (?)
}
```

---

## Leaderboard \[Users Example] (GET request)
Description can be implemented as a popupto show info for example about courses which teaches this professor.

Response:
```json
[
  {
    "nickname": "",
    "rating": "",
    "photo": (?),
    "description": (?)
  }
]
```

---
