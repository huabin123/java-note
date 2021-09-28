# 概述

- Spring MVC，本质上相当于Serverlet。

- Spring MVC是结构最清晰的Servlet+JSP+JavaBean的实现。

- Spring MVC有许多可插拔组件，具有高度可配性。

- Spring MVC中，Controller代替Servlet来担负控制器的职责，用于接收请求，调用相应的Model进行处理，处理器完成业务处理后返回处理结果。Controller调用相应的View并对处理结果进行视图渲染，最终客户端得到响应信息。