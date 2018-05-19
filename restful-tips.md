三个要点：
1. 用名词来表示资源；比如 /url/reports, /url/notes, /url/payment等
2. 用http method来表示动作；POST用于新建资源, PUT用于更新资源，GET用于查询，DELETE用于删除;
   对于POST和PUT也可以从该操作是否幂等来理解，如果不是幂等的，就用POST，否则可以考虑使用PUT；这个也是浏览器会进行的操作，如果一个PUT操作失败了，浏览器可能会重新发起该请求；
   还有一个角度，当往一个集合中，添加一个新的资源时，用POST；当确定更新一个资源时，用PUT；举例来说

   ```
     POST .../blogs  {title: xxx, id: yyy, description: zzz}
     PUT  .../blog/{id} {title: xxx, description: www}
   ```
3. 用http code来标示操作的结果，常见20x表示成功，500表示异常，401表示权限问题等；