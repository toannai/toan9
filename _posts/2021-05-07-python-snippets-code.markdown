---
layout: post
title: Snippet code python hay sử dụng,
date: 2021-05-07 00:32:20 +0700
description: Snippet code python hay sử dụng, # Add post description (optional)
img: 2021/05/07/210507_snippet_code.jpg
fig-caption: # Add figcaption (optional)
tags: [Python]
---

Một số snippet code python rất hay sử dụng, mỗi lần viết lại phải đi tìm kiếm hết hơi do vậy chúng sẽ được đặt tại đây, khi cần lấy ra và xài thôi.

#### 1. Đọc file line by line
Đọc một lần
```python
myfile = open("test.txt", "r")
for line in myfile:
    print(line)
myfile.close()
```
Đọc nhiều lần
```python
myfile = open("test.txt", "r")
while myfile:
    line  = myfile.readline()
    print(line)
    if line == "":
        break
myfile.close()
```

#### 2. Send HTTP request
GET request
```python
# import requests module
import requests
# Making a get request
response = requests.get('https://api.github.com/') 
# print response
print(response)
# print request status_code
print(response.status_code)
```
POST request
```python
# import requests module
import requests
data = {"username": username, "password": password}
# Making a get request
resp = requests.post('https://api.github.com/', data=data)
# print response
print(resp)
# print request status_code
print(resp.status_code) 
```