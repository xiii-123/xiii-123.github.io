---
title: axios入门
published: 2025-03-20
description: 'axios的基本使用'
image: ''
tags: [axios]
category: 'java-web'
draft: false 
---

Axios是一个基于Promise的HTTP客户端，用于node.js和浏览器中发送请求。它支持多种请求方式，包括GET、POST、PUT、DELETE等，并且可以自动转换请求和响应数据。
## 基本用法
1. **安装Axios**：
   ```bash
   npm install axios
   ```
2. **引入Axios**：
   ```javascript
   import axios from 'axios';
   ```
3. **发送GET请求**：
   ```javascript
   axios.get('https://api.example.com/data')
     .then(response => {
       console.log(response.data);
     })
     .catch(error => {
       console.error('Error fetching data: ', error);
     });
   ```
4. **发送POST请求**：
   ```javascript
   axios.post('https://api.example.com/data', {
       key: 'value'
     })
     .then(response => {
       console.log(response.data);
     })
     .catch(error => {
       console.error('Error posting data: ', error);
     });
   ```
## 常见方法
- `axios.get(url, [config])`：发送GET请求。
- `axios.post(url, data, [config])`：发送POST请求。
- `axios.put(url, data, [config])`：发送PUT请求。
- `axios.delete(url, [config])`：发送DELETE请求。
- `axios.all(iterable)`：并发执行多个请求。
- `axiosspread(callback)`：用来处理axios.all的结果。
- `axios.create([config])`：创建一个新的axios实例。
## 最佳实践案例：
**案例：使用Axios实现用户登录**
1. 用户在登录表单中输入用户名和密码。
2. 表单提交时，使用Axios发送POST请求到服务器。
3. 服务器验证用户信息，返回token。
4. 使用返回的token进行后续请求的认证。
代码示例：
```javascript
// 用户登录函数
function userLogin(username, password) {
  axios.post('https://api.example.com/login', {
    username: username,
    password: password
  })
  .then(response => {
    const token = response.data.token;
    localStorage.setItem('userToken', token); // 存储token
    console.log('Login successful!');
  })
  .catch(error => {
    console.error('Login failed: ', error);
  });
}
```
## 同步
`async`和`await`，这两个关键字在现代JavaScript中用于处理异步操作，使得代码更加简洁和易于理解。下面是结合`async`和`await`使用Axios的示例：
### 基本用法 with `async`/`await`：
1. **发送GET请求：**
   ```javascript
   async function fetchData() {
     try {
       const response = await axios.get('https://api.example.com/data');
       console.log(response.data);
     } catch (error) {
       console.error('Error fetching data: ', error);
     }
   }
   ```
2. **发送POST请求：**
   ```javascript
   async function postData() {
     try {
       const response = await axios.post('https://api.example.com/data', {
         key: 'value'
       });
       console.log(response.data);
     } catch (error) {
       console.error('Error posting data: ', error);
     }
   }
   ```
### 最佳实践案例：
**登录并获取Token：**
```javascript
async function login(username, password) {
  try {
    const response = await axios.post('https://api.example.com/login', {
      username: username,
      password: password
    });
    const token = response.data.token;
    localStorage.setItem('userToken', token); // 存储token
    console.log('Login successful!');
  } catch (error) {
    console.error('Login failed: ', error);
  }
}
```
**使用Token进行认证的请求：**
```javascript
async function fetchProtectedData() {
  try {
    const token = localStorage.getItem('userToken');
    const response = await axios.get('https://api.example.com/protected', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });
    console.log(response.data);
  } catch (error) {
    console.error('Error fetching protected data: ', error);
  }
}
```
### 注意事项：
1. **错误处理：** 使用`try...catch`语句来捕获和处理错误。
2. **异步函数：** 确保使用`async`关键字定义异步函数。
3. **等待响应：** 使用`await`关键字等待异步操作的完成。
4. **避免阻塞：** 不要在顶层作用域或其他非异步函数中使用`await`，以避免阻塞事件循环。
5. **并发请求：** 如果需要同时发送多个请求，可以使用`Promise.all`来处理。
6. **超时设置：** 可以通过配置来设置请求超时时间。
7. **取消请求：** 使用`axios.cancelToken`来取消未完成的请求。
通过结合`async`和`await`使用Axios，可以编写更加清晰和易于维护的异步代码。
