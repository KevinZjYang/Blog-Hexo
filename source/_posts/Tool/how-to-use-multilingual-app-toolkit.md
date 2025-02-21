---
title: 如何使用微软多语言翻译工具Multilingual App Toolkit
tags:
  - UWP
categories:
  - - 开发笔记
date: 2018-10-23 11:05:19
---

多语言支持是一个枯燥乏味的工作，加之自身水平限制。使用微软的Multilingual App Toolkit就能在一定程度上解决这个问题。刚开始使用这个工具的时候完全不会，安装好后发现无法翻译。曾几何时还以为是软件的锅，直到前一段时间，我才发现了正确的安装方式。因此在这里记录下来。

## 1、准备工作。

*   Multilingual App Toolkit v4.0(VS 2017+) (VS扩展里面搜索就可以下载）
*   [Microsoft Azure 的免费翻译服务](https://portal.azure.com/)  
*   [Multilingual app toolkit 4.0 Editor](https://developer.microsoft.com/en-us/windows/develop/multilingual-app-toolkit)

## 2、安装和设置

*   关闭VS2017,安装下载好的“Multilingual App Toolkit v4.0 (VS 2017)”安装包，一路默认安装即可。
*   点击小娜，打开“凭据管理器”。

![](https://5qgppg.sn.files.1drv.com/y4m4dFz_EDLZhDcPy9vR_t49391XadLSJbHhH1-cCR-1jrVbgfdfz-fT4THsPKXJRH9fbFnNYM-V3MQBcHwiq72He5XfzIqHQKAbaCRd5sc7wSOZPsLLyXrfruUU2YA13LUH2QAZKnGnVQt5oswHHnsBqMu4QMuu9rRScVr2yT1nqz6V3kDXvaiHcikZrGhdWMb9sRdoWuSYs9H7Yn-Ft7VCg?width=398&height=645&cropmode=none)

打开“凭据管理器”

![](https://hiv4gw.sn.files.1drv.com/y4mBr1iJ7erfOoIwCX6FWRKe6R8DTLsyE_OY3mazzyBZje2b6VSmwHiD0W_6jtwm8uWQj8ppBAqo_olFZToZlwBwU2X5fxRtMY6oEDYpA4w-llaLbkGPjGXHOsi3gSsS6cIUypvCtpjbR9idZCo2y-5FIf4l503QJGTbRJIHYgxHkYdinBoel78FMxgD6rDu_FAGp6sfOVQ0lAHvdYDERtlNw?width=947&height=694&cropmode=none)

*   选择“Windows 凭据”，添加“普通凭据”。

![](https://31pkeq.sn.files.1drv.com/y4mIiEm1k_c6mprCOFSmeXqz5a3Dy2HsrQfPWbUnE5l0oBIKjiGLxYCIfwASiDqlXcNfHaq4x944Z8CoGqRCVaZdzo8gMGkmWrgOQVkULnycEdIE-HRifX1fs47ftN7IwmcbTvHGUejLXOCfWryZbQxlwlBBNu-tdS0vNbIrB40aeORWw2WRSQKDjXNPLeNCFtLIiH5xJ8Yevv400vZP444MA?width=1095&height=855&cropmode=none)

*   第一栏输入：**Multilingual/MicrosoftTranslator**
*   第二栏输入：**Multilingual App Toolkit**
*   第三栏输入：（Microsoft Azure的免费翻译api的密钥）
*   注意前两栏前后不能有空格。点确定以后就大功告成了。

## 3、启用多语言应用程序工具包。

必须先为项目启用Multilingual App Toolkit，然后才能开始本地化应用程序。

要启用Multilingual App Toolkit：

*   打开项目解决方案文件。
*   在解决方案资源管理器中选择所需的项目。
*   在“工具”菜单上，选择“启用Multilingual App Toolkit”。

现在就可以正常使用了。

![](https://es8lta.sn.files.1drv.com/y4mQyM_SxTylUsIGTe3MbtbsF44yRh__-q9eIXTlwOcxuRvmFYeBNUplGQjsLSLspA-epvOUQfL_KsAljYAFfaseFHhg4-oak_L3-qJPD2mDXxCZV2pZA-XipjG8ksXZXmwDyyvRsFGUCovjkirqvYAeraw3mkxe0-7kwaROVGsBzuXDQNWA_KnCf-kp67mG-s7UCRNhIlv3zetR2qbQpvy7g?width=1059&height=210&cropmode=none)

## 附：如何获取免费的Microsoft Azure翻译密钥

*   登陆[Microsoft Azure](https://portal.azure.com)，搜索“免费”。然后找到“翻译工具文本API”。可以免费使用2000000个字符。

![](https://fuszvg.sn.files.1drv.com/y4mu_gKtr9sBFY4LhyaECHTw_RNLxf0NboWS5JXKXxsCr9fIdJ3pJYI-e_KShiQy1P5yqix1-wPWIUS264jBM69HmZmoK9hMO2duSWMhwbzYvn8O1TL7yj27mNSWWOnO49_P-ARmLJ-7YuFsrN0ZiWXse775ms2nJsuikGIxR7LzoGLVaIgaU7_VrrtDdSH1UvITSvih9HepvKK2EB0mDJ1Ug?width=1186&height=757&cropmode=none)

*   按照提示创建，定价选f0，其他随便。创建完成后。打开，就可以看到密钥了，有两个密钥，选择任何一个都可以。

![](https://slhapq.sn.files.1drv.com/y4mx7LdDQIoxStIMB23eNne0xArJHa3zagLNTy5ZCbtmL_Ndb6WOKaz33RfKCRuSFTQi6xDZYkmuk31iO1v_bqAw1UQWJ8agxbeWGOvtoxe-Z93t0e23sy3PfKaCHa2-qOsnJw2Y6rVTl4uDkJizrcvsbVSCIGIxG-U4kEbB9wCf4tpsbDpWb13Yw4oyd3710FFIWDsz75uY_OO69Cig330Zg?width=1121&height=745&cropmode=none)

![](https://sy0cna.sn.files.1drv.com/y4mDuJuLiPR30WBqh8Yq5V-2OUL0IuZf0qwwkyFKZT2r_mltdcuW6F8jpKwvWt5dhdPSIIEooq9vdm0JQwAdQQGMV6i0EjVhluBC-PZ2vuWJJ5uV6rxejC-zeNfaJKPxoAnpszs2-jUpfW1ObVyBuXM8mKS8ghhZWAL5FoU7LVszW8TMMJyP7jT7DHbOWbQvdjfcNpRC_tzaW7aGjpvn3cP3Q?width=1920&height=1034&cropmode=none)