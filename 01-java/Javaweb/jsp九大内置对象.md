#### 内置对象

1. pageContext
2. request
3. session
4. application
5. request
6. page
7. out
8. config
9. exception



#### 介绍

| 变量名      | 真实类型           | 作用                                                         |
| ----------- | ------------------ | ------------------------------------------------------------ |
| pageContext | PageContext        | 当前页面的共享数据，还可以获得其他八个内置对象（并且可以通过它转换成其他几种对象） |
| request     | HttpServletRequest | 一次请求多个资源（转发）                                     |
| session     | HttpSession        | 一次回话的多个请求间                                         |
| application | ServletContext     | 所有用户之间共享的duixang                                    |
| request     | HttpServletRequest | 响应对象                                                     |
| page        | Object             | 当前页面的（servlet）的对象  this                            |
| out         | JspWrite           | 输出对象，将数据输出到页面上                                 |
| config      | ServletConfig      | Servlet的配置对象                                            |
| exception   | Throwable          | 异常对象                                                     |

