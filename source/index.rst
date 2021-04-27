.. CITA-Cloud documentation master file, created by
   sphinx-quickstart on Wed Nov  4 12:01:45 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

CITA-Cloud 文档
======================================
.. image:: _static/images/cita-cloud-log.png

CITA-Cloud 是一个基于云原生的联盟链框架，用户可以基于该框架快速定制出一个适合具体业务场景的链。其主要特性有：

    - 语言无关。
        整个框架采用微服务架构，不同微服务可以采用不同的编程语言。各种语言的开发者都可以方便的参与。也可以在开发过程中选择最适合的语言，方便复用已有的软件栈或者库。
    - 组件解耦。
        微服务之间高度解耦，每个微服务可以有多种不同的实现，相互之间可以随意替换。可以形成组件市场。用户根据需要选择适合的组件，组成一条自定义的链。场景变化时，也可以快速切换其他的组件。
    - 场景定制。
        如果已有的组件都不能满足用户需求，可以针对场景定制某个组件。同时可以复用其他组件，达到快速定制的目标。
    - 兼容。
        得益于灵活的架构，可以复用已有区块链产品的组件，达到兼容其其生态的目的。也可以通过跨链的方式与其他区块链系统协作。

- `Github主页 <https://github.com/cita-cloud>`_
- `官方网站 <https://www.citahub.com/#/cita-cloud>`_
- `技术论坛 <https://talk.citahub.com/c/24-category/24>`_
- `rfcs <https://github.com/cita-cloud/rfcs>`_

.. toctree::
   :hidden:
   :maxdepth: 2

   introduction
   getting-start
   configuration-guide
   deployment-guide
   upgrade
   node-management
   privacy
   governance
   data-managment
   ops
   architecture
   rpc
   release
   components
   faq
   glossary
      
