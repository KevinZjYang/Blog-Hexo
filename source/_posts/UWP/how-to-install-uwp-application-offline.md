---
title: 如何离线安装UWP应用
tags:
  - PowerShell
  - UWP
  - Win10
categories:
  - - 开发笔记
date: 2019-03-15 15:53:57
---

商店经常性会抽风，好多用户反映无法在商店下载和安装应用。这我也没办法啊，有的用户就回去打1星，然后描述为“垃圾软件，根本无法下载和安装”。真的很无奈，主动联系我的，我就发给他们离线包，但是好多人是不会怎么安装的，所以写这篇文章，记录一下具体的安装过程。

## 方法一：直接安装

**首先去设置-更新和安全-开发者选项-开启旁加载应用（可以参考方法2中开启开发者模式）**

### 1、确认证书是否正确安装

打开安装包内`.msixbundle` 后缀的文件

![](https://22qyaa.sn.files.1drv.com/y4mEprkomj044HjG_rrIbnOw3IoFlkSsEzPegLxfL29YOKy2y4uDH-w88FdUJCRjT2Ax1OcZGR0UldodiufqPgXwdD0NYnG7KA8GXsZxvLkaxYe2b2Neq2FBAFEtu-wb2tgPAplr-BISMNezzB9hi0M4xw14PVteC2G_i6G6D-amO0W_IYR8RD7idfCOFyY0VQyjor6xbUyJfljE4asRUc5qw?width=822&height=329&cropmode=none)

如果显示如下图所示，点安装等待完成即可。

![](https://evtnrq.sn.files.1drv.com/y4m_IyT6PlsJv20fBqV5Lj5BtcqnPzS7nZi7WPpJmxwg7inzId-X1arPItvDTqkCHdKLy_OCT6Ke9p9735NEmPIA6K0Tk_MEdV_7DnpzyyMpADgYe65ozNbt_sebYXDiH-0TYHMmbQkIFoKPjc_yhTYzLPiAGGiWZFva87GSkQ6eqUKnLCruuSBsObXo1Pq_Bxz6hLlwAt_jcyW29a-KWVB4g?width=650&height=415&cropmode=none)

### 2、证书未安装

![](https://rmio6a.sn.files.1drv.com/y4moZG-tv0z6k5YarjyJ3cP-ka8K88QRJRpvxdcEYBFbPYYJ_UQ1CWCFmkZL3QI8NXQSOYFsc46GWKyWUmJgJK4dP1RlYT5QlsVy1XCOd1C-lLgW2XjX1JGnfkX1d2wwAU4FbwjQa7ThRCEacJfVncN7gzep1J33_S9831ola8KRN1GYSbhzFPXwvJp9ubgbr2CSArlzeWdGP5Tqb6jQ3jEcw?width=657&height=414&cropmode=none)

右键安装包内`.cer` 后缀的文件，选择安装证书

![](https://hpjuoa.sn.files.1drv.com/y4mKqGLFq9p0sde-pQvRpCoKoaw9o5AzXB7yxvzERu0fbox_kLWu3QU-0kTMU2NGVQ2LY4hSMmObnKdInWoK6BJwFCGopF6b8RmLPhYdmmhe0ErAqnj791JcX81O5JSxCNsLUNdIghfF5aLtF2v-vyfZimQSNTQb_QBsYHHmRrWy5Greuzzm91TnbI8eyME355bn7-38XVDEvx71wScBwM-Wg?width=800&height=388&cropmode=none)

选择当前用户

![](https://ulywqw.sn.files.1drv.com/y4m_xRDe9ibuKuE-XZOP4wD5c7m-cxWjlxgSDx3mbdo0Bll-S2uVjfraxXpz8cs1yqF89stxCoyHt0aC8XBPcTqsK4mHyX8PFieIqIWYlzBCzMJz3dtAtMhgS-tlFGgkjuOheLCnG8m0xGxJqadEZFJhOc86GTtoNmhIiGXHnN-oEwohX-PCN11esLazvl75OVwz95vTbMA094HO3O7m0IhPg?width=615&height=635&cropmode=none)

按照下图说明操作

![](https://pyfjuw.sn.files.1drv.com/y4mnsGRdowTZRNcu-OjREjBm2VBUkX9BrlGC1M41RAuV2_mqFqMgaoREGhv8W_eMIcOXBg6p7qamABZD9MVYhfhcKdmwX4RDztN32sc_BAEbMI36j4YTtwMvOJTH1J0iRh8VWB7jJaK5Q08TEQm8HdDqnuB5IaySiwr8q66S5JQh9BwgyQM5j-x8dOV2H1qY936IvsdVwI-mNW-iqEQIK33og?width=914&height=631&cropmode=none)

然后点“完成”，提示导入成功，就完成了。重复1，继续安装应用。

## 方法二：使用PowerShell

### 1、开启开发者模式

进入设置-更新和安全-开发者选项，打开开发者模式。

![](https://jtjtmq.sn.files.1drv.com/y4mn5xXQQrIF_6_TMvwGx3ZYTRypEqkU4SjlxuRMJeNfuFlkkpVtWMvrwMtT8kZvI0qGZyevmzD6N4Tr0RwOw0RqbVNZrd2Aw9vD50zYT7EYxSdTNK0FHrY-KafpdY-7ruMjmnUqaP0Nzb9Qjhnuo7FGsi1ZKgcbUUHbb7PuZCwUhUw2Af3omVuw92SvVxsryEP2GRyz2yQiAtZEofLE6QsuQ?width=1216&height=941&cropmode=none)

### 2、使用PowerShell安装

找到安装包的“Add-AppDevPackage.ps1”，然后右键选择“使用PowerShell运行”。

![](https://wub7ow.sn.files.1drv.com/y4m0BYtiWZzKqwHCATUBjpP5z4NOieSW05-VN8S7Boie0TkQUsv4XoyIb-fVuMYXlakiz-1WOOkmV8bUDw10GWpXfzAeVzTGlk5z1c7L88Png81TTA3TuCOfzQEmJJHntfT6c7gwgeN6Hy9GUw1KPfWsT2Nkbz2pyltxcLZbhSbY5apGnd5krn9TtD5-MTCfrIki-dMXHiuLg3ijmbwMNNkqQ?width=709&height=387&cropmode=none)

![](https://ts0rha.sn.files.1drv.com/y4mJExxPtV4VN8zptn5FRnmce6iMruQoaRInsYk-kYC6-VuVt2sGlX2IndEbY_9fkrrCdYrCtkci-2I7faxM0r51gPRt_wf3iAUkrl_pGPRgobTKxb3yQeDXa04CsExNLQ7G8jaLWN_saEN7rk35uXm_-8AVRnycE_RmVTq3mi77Waq9KhEWzA_mj5IqNCm9xYzF7xHfeIy5lGJonQtMNLXvg?width=717&height=457&cropmode=none)

### 3、执行策略更改

提示需要修改执行策略，此处输入“A”，也就是全是\[A\]，然后回车。以后再安装就不需要修改这个了。

![](https://hbzoyq.sn.files.1drv.com/y4mg0UZQ2yz9s0yDBIGpHr2dJyQ23bgvgI7giAxvzhNNyg5DXLtMhh4is60o5mnN-EeCrVlxYIT8LtV-BF67WfEuXxzjqJ1X45P99MYGghQiZfppSt8c1WFmCdklSCRgEL_5M4vnUQ2QZZ9qPuXRmsEwbayJMjJTqxmWXVAmedXMIc-CooGmn1OYrNI9-RMd-vW1Lr95Q2muS6H0p7gzwSYNA?width=1972&height=1556&cropmode=none)

![](https://h4spba.sn.files.1drv.com/y4mn6MK5npB3-2W_-uN0EhiYraIttoUdsFModNcyNLQXBQYPnFFzyp-2-ZhdvLy0_cIsJVkc72se5O74fSIC63zSuE-thMBJBm5346zT7NU6RViX_LpFoNVuz7eq61jcItF6hbZPnS5OiytkBYqIXoUqP-fCF99ltWACw1dfoPnxNVBnuqUbQELdkiRIcTaUCQJpe1PZ6FAfqV_J2Lnepv5wA?width=1964&height=1560&cropmode=none)

### 4、安装签名证书

提示是否继续。此处选是，也就是输入“Y”，然后回车。此处会弹出账户控制，选是，即可安装证书。

![](https://4lp7pq.sn.files.1drv.com/y4muDm3W92I9beO18DIlqTiWbUOgCS95TtR22zlYg2p7asNKLjdYd_gXxw9VR8RRmkidvxfnX1AdUcig07ox5_mPWwTjTC__NGFMwaVIx3PiyeefYy6P-HbCn5E-5HBF-BYVEhEjRLj1IQIclLxOUdalDLnkKn6KxS3sj1j69R8ccxfCoviRxOpewFXy2ptcAA5YAE0uCV5x4a7FJ_yZHnbmQ?width=1974&height=880&cropmode=none)

![](https://s7hw4q.sn.files.1drv.com/y4mqS7GVMV8Xn9EXyUz0kumLBU1v6jBOuAj-vus0jBXa_jkrOOC-kE9dARm5pwWUXKH1KP15N-T0f87JVK13kpatBOs04cdOpRISI2kFRI-Hm9ClLWk5SsebT0BPKjOcCNnJH6oRoHq7c8NrodL2DRwTI3SNg9hdpG388J8xx2bAIEt0Ca6dy5rBQKNdJTi4_68YFdd0W1s-Xm5sRtejV521Q?width=4032&height=3024&cropmode=none)

### 5、安装成功

按回车结束，就完成了。  
PS：文中截图不是一次截的，所以安装包路径会不同，无所谓了。

![](https://getkpg.sn.files.1drv.com/y4mvta_8bVI1h5_H-RTi_u1ahhjyRcKq2Ne5digY7xCVrAwRnvQplS-XwHjdoZiNIaFTCbYLKIXYsZxAZU3YqNdj9hoMEuG3TlVClZNS-eaGSD9cD2WRHAKsE0nyrSAvn2cyoohC7-LcnFH8EJejdQbP6nkjczh528-Z9dFPE81r1oN5zsikxdsu61VE-RGHq33Hwp03kcmCEHhfQ257Kp64Q?width=1216&height=928&cropmode=none)