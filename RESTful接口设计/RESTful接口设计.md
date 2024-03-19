# RESTful 接口设计

1. **动作**
   RESTful 的接口设计使用的 HTTP请求只有五种
   - GET : 查询 读取资源
   - POST : 创建资源
   - PUT : 更新全部资源
   - DELETE : 删除资源
   - PATCH : 更新部分资源
2. **资源**
   一个URL代表一种资源，使用名词来表示资源，使用复数名词来说明资源的集合性质。
3. **状态码**
   API 使用到的状态码只有 2XX, 4XX, 5XX三种。
   - 2xx状态码 表示操作成功

       ```txt
       200 GET OK
       201 POST Created
       200 PUT OK
       200 PATCH OK
       204 DELETE No Content       
       ```

   - 4xx状态码 表示客户端错误

       ```txt
       400 Bad Request：服务器不理解客户端的请求，未做任何处理。
       401 Unauthorized：用户未提供身份验证凭据，或者没有通过身份验证。
       403 Forbidden：用户通过了身份验证，但是不具有访问资源所需的权限。
       404 Not Found：所请求的资源不存在，或不可用。
       405 Method Not Allowed：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
       410 Gone：所请求的资源已从这个地址转移，不再可用。
       415 Unsupported Media Type：客户端要求的返回格式不支持。比如，客户端要求返回XML格式，API只能返回JSON格式。
       422 Unprocessable Entity ：客户端上传的附件无法处理，导致请求失败。
       429 Too Many Requests：客户端的请求次数超过限额。
       ```

   - 5xx状态码 表示客户端错误

       ```txt
       500 Internal Server Error：客户端请求有效，服务器处理时发生了意外。
       503 Service Unavailable：服务器无法处理请求，一般用于网站维护状态。       
       ```

4. **例外**
   - 使用 POST，为需要的动作增加一个 endpoint，使用 POST 来执行动作，比如: POST /resend 重新发送邮件。

   - 增加控制参数，添加动作相关的参数，通过修改参数来控制动作。比如一个博客网站，会有把写好的文章“发布”的功能，可以用上面的 POST /articles/{:id}/publish 方法，也可以在文章中增加 published:boolean 字段，发布的时候就是更新该字段 PUT /articles/{:id}?published=true

   - 把动作转换成资源，把动作转换成可以执行 CRUD 操作的资源， github 就是用了这种方法。

[***RESTful 接口的后端实现***](./RESTful接口的后端实现.md)
